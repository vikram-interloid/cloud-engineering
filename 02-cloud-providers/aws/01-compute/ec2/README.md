# Amazon EC2 for Web Development

> **Focus**: Production web server setup, Nginx reverse proxy, systemd process management, zero-downtime rolling updates, and stateless session scaling.

---

## 1. Production Host Architecture (Nginx + Node/Python Backend)

A production EC2 web instance runs **Nginx** as a high-performance reverse proxy for TLS termination, gzip compression, and static asset caching, forwarding dynamic requests to internal web application processes via Unix sockets or localhost ports.

```mermaid
flowchart TD
    ALB["Application Load Balancer<br/>(Health Checks on :80)"] -->|HTTP Forward| NGINX["<b>Nginx Reverse Proxy</b> (:80 / :443)"]

    subgraph EC2_INSTANCE["Amazon EC2 Web Host"]
        direction TB
        NGINX -->|Proxy Pass /api/| SOCK["Unix Domain Socket / Localhost :3000"]
        NGINX -->|Static Files /public/| DISK["Local NVMe EBS Cache"]
        
        subgraph PROCESS_MGR["systemd Process Supervision"]
            SOCK --> APP1["Node.js / Python Gunicorn Worker 1"]
            SOCK --> APP2["Node.js / Python Gunicorn Worker 2"]
            SOCK --> APP3["Node.js / Python Gunicorn Worker 3"]
        end
        
        APP1 -.-> IAM_CREDS["IAM Instance Profile (Auto-rotated STS tokens)"]
    end

    APP1 -->|Shared State| REDIS[("Amazon ElastiCache Redis<br/>(User Sessions & Rate Limits)")]
    APP1 -->|Persistence| RDS[("Amazon Aurora PostgreSQL")]

    style ALB fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style EC2_INSTANCE fill:#10b981,fill-opacity:0.06,stroke:#10b981,stroke-width:2px,color:inherit
    style NGINX fill:#10b981,fill-opacity:0.2,stroke:#10b981,stroke-width:1.5px,color:inherit
    style PROCESS_MGR fill:#f59e0b,fill-opacity:0.08,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style REDIS fill:#ef4444,fill-opacity:0.15,stroke:#ef4444,stroke-width:1.5px,color:inherit
    style RDS fill:#3b82f6,fill-opacity:0.15,stroke:#3b82f6,stroke-width:1.5px,color:inherit
```

---

## 2. Zero-Downtime Rolling Deployment with ALB

When deploying new code across an Auto Scaling Group (ASG), the Application Load Balancer coordinates with **Connection Draining (Deregistration Delay)** so active HTTP requests finish before instances terminate.

```mermaid
sequenceDiagram
    autonumber
    participant ASG as Auto Scaling Group
    participant ALB as Application Load Balancer
    participant OldEC2 as v1 Instance (Old)
    participant NewEC2 as v2 Instance (New)

    Note over ASG: Deployment Triggered (New Launch Template)
    ASG->>NewEC2: Launch v2 Instance & Run UserData Boot
    NewEC2->>ALB: Register with Target Group
    loop ALB Health Checks
        ALB->>NewEC2: GET /health (Port 80)
        NewEC2-->>ALB: 200 OK (Healthy)
    end
    ALB->>NewEC2: Shift 50% Live Web Traffic to v2
    
    rect rgb(239, 68, 68, 0.1)
    Note over ALB,OldEC2: CONNECTION DRAINING (e.g. 60 seconds)
    ALB->>OldEC2: Stop sending NEW requests
    OldEC2-->>ALB: Complete in-flight HTTP requests
    end
    
    ASG->>OldEC2: Terminate v1 Instance
    Note over ALB,NewEC2: 100% Traffic running on v2 with ZERO dropped packets!
```

---

## 3. Stateless Web Tier vs Sticky Sessions

```mermaid
flowchart LR
    subgraph BAD["Anti-Pattern: Sticky Local Sessions"]
        direction TB
        B_ALB["ALB (Sticky Cookie)"] --> B_EC1["EC2 A: Session Memory"]
        B_ALB --> B_EC2["EC2 B: Session Memory"]
        Note1["If EC2 A crashes, users are logged out!"]
    end

    subgraph GOOD["Production Pattern: Stateless with Redis"]
        direction TB
        G_ALB["ALB (Round Robin)"] --> G_EC1["EC2 A (Stateless)"]
        G_ALB --> G_EC2["EC2 B (Stateless)"]
        G_EC1 --> REDIS_HUB[("ElastiCache Redis Session Store")]
        G_EC2 --> REDIS_HUB
        Note2["Any instance can crash or scale with zero session loss!"]
    end

    style BAD fill:#ef4444,fill-opacity:0.08,stroke:#ef4444,stroke-width:1.5px,color:inherit
    style GOOD fill:#10b981,fill-opacity:0.08,stroke:#10b981,stroke-width:2px,color:inherit
```

---

## 4. Production Web Server User Data Script

```bash
#!/bin/bash
# Amazon Linux 2023 - Web Server Automated Bootstrap
dnf update -y
dnf install -y nginx git nodejs

# Configure Nginx Reverse Proxy
cat << 'CONF' > /etc/nginx/conf.d/webapp.conf
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
    }
}
CONF

systemctl restart nginx
systemctl enable nginx
```

---

## 5. Infrastructure as Code & SDK Implementations

### CDK (TypeScript)

```typescript
import * as cdk from 'aws-cdk-lib';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import { Construct } from 'constructs';

export class IaaSStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const vpc = ec2.Vpc.fromLookup(this, 'DefaultVpc', { isDefault: true });

    const securityGroup = new ec2.SecurityGroup(this, 'WebServerSG', {
      vpc,
      description: 'Allow HTTP and SSH',
      allowAllOutbound: true,
    });
    securityGroup.addIngressRule(ec2.Peer.anyIpv4(), ec2.Port.tcp(80), 'Allow HTTP');
    securityGroup.addIngressRule(ec2.Peer.anyIpv4(), ec2.Port.tcp(22), 'Allow SSH');

    new ec2.Instance(this, 'ComputeWorker', {
      vpc,
      instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
      machineImage: ec2.MachineImage.latestAmazonLinux2023(),
      securityGroup,
      blockDevices: [
        {
          deviceName: '/dev/xvda',
          volume: ec2.BlockDeviceVolume.ebs(20, {
            volumeType: ec2.EbsDeviceVolumeType.GP3,
            encrypted: true,
          }),
        },
      ],
    });
  }
}
```

### CDK (Python)

```python
import aws_cdk as cdk
from aws_cdk import aws_ec2 as ec2
from constructs import Construct

class IaaSStack(cdk.Stack):
    def __init__(self, scope: Construct, construct_id: str, **kwargs) -> None:
        super().__init__(scope, construct_id, **kwargs)

        vpc = ec2.Vpc.from_lookup(self, "DefaultVpc", is_default=True)

        security_group = ec2.SecurityGroup(
            self, "WebServerSG",
            vpc=vpc,
            description="Allow HTTP and SSH",
            allow_all_outbound=True
        )
        security_group.add_ingress_rule(ec2.Peer.any_ipv4(), ec2.Port.tcp(80), "Allow HTTP")
        security_group.add_ingress_rule(ec2.Peer.any_ipv4(), ec2.Port.tcp(22), "Allow SSH")

        ec2.Instance(
            self, "ComputeWorker",
            vpc=vpc,
            instance_type=ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
            machine_image=ec2.MachineImage.latest_amazon_linux2023(),
            security_group=security_group,
            block_devices=[
                ec2.BlockDevice(
                    device_name="/dev/xvda",
                    volume=ec2.BlockDeviceVolume.ebs(
                        volume_size=20,
                        volume_type=ec2.EbsDeviceVolumeType.GP3,
                        encrypted=True
                    )
                )
            ]
        )
```

### YAML (CloudFormation)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Amazon EC2 Instance with GP3 EBS Volume and Security Group

Parameters:
  VpcId:
    Type: AWS::EC2::VPC::Id
  SubnetId:
    Type: AWS::EC2::Subnet::Id

Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP and SSH
      VpcId: !Ref VpcId
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0

  ComputeWorkerInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro
      ImageId: ami-0c101f26f147fa7fd
      SubnetId: !Ref SubnetId
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup
      BlockDeviceMappings:
        - DeviceName: /dev/xvda
          Ebs:
            VolumeSize: 20
            VolumeType: gp3
            Encrypted: true
```

### SDK (TypeScript)

```typescript
import { EC2Client, RunInstancesCommand } from '@aws-sdk/client-ec2';

const client = new EC2Client({ region: 'us-east-1' });

async function launchInstance() {
  const command = new RunInstancesCommand({
    ImageId: 'ami-0c101f26f147fa7fd',
    InstanceType: 't3.micro',
    MinCount: 1,
    MaxCount: 1,
    SecurityGroupIds: ['sg-0123456789abcdef0'],
    SubnetId: 'subnet-0123456789abcdef0',
    BlockDeviceMappings: [
      {
        DeviceName: '/dev/xvda',
        Ebs: {
          VolumeSize: 20,
          VolumeType: 'gp3',
          Encrypted: true,
        },
      },
    ],
    TagSpecifications: [
      {
        ResourceType: 'instance',
        Tags: [{ Key: 'Name', Value: 'ComputeWorker' }],
      },
    ],
  });

  const response = await client.send(command);
  console.log('Instance ID:', response.Instances?.[0].InstanceId);
}

launchInstance();
```

### SDK (Python)

```python
import boto3

ec2_client = boto3.client('ec2', region_name='us-east-1')

def launch_instance():
    response = ec2_client.run_instances(
        ImageId='ami-0c101f26f147fa7fd',
        InstanceType='t3.micro',
        MinCount=1,
        MaxCount=1,
        SecurityGroupIds=['sg-0123456789abcdef0'],
        SubnetId='subnet-0123456789abcdef0',
        BlockDeviceMappings=[
            {
                'DeviceName': '/dev/xvda',
                'Ebs': {
                    'VolumeSize': 20,
                    'VolumeType': 'gp3',
                    'Encrypted': True
                }
            }
        ],
        TagSpecifications=[
            {
                'ResourceType': 'instance',
                'Tags': [{'Key': 'Name', 'Value': 'ComputeWorker'}]
            }
        ]
    )
    print("Launched Instance ID:", response['Instances'][0]['InstanceId'])

launch_instance()
```
