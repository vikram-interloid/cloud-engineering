# AWS CloudFormation for Web Development

> **Focus**: Modular Infrastructure as Code (IaC), cross-stack references (`Fn::ImportValue`), Change Sets for safe releases, and protecting stateful databases.

---

## 1. Modular Multi-Stack Architecture for Web Applications

Enterprise web projects avoid giant monolithic templates by decomposing infrastructure into lifecycle-based stacks:

```mermaid
flowchart TD
    subgraph STACK_NET["1. Network Stack (Slow Churn - VPC, Subnets, NAT)"]
        VPC["VPC: 10.0.0.0/16"]
        PUB_SUBS["Public Subnets"]
        PRIV_SUBS["Private Subnets"]
        VPC --> PUB_SUBS
        VPC --> PRIV_SUBS
    end

    subgraph STACK_DATA["2. Data Stack (Stateful - Aurora / DynamoDB)"]
        AURORA["Aurora PostgreSQL Cluster"]
        DDB["DynamoDB Tables"]
        S3_UPLOADS["User Uploads S3 Bucket"]
    end

    subgraph STACK_APP["3. Application Stack (High Churn - ALB, ECS / Lambda)"]
        ALB["Application Load Balancer"]
        ECS_SVC["ECS Fargate Web Service"]
        ALB --> ECS_SVC
    end

    STACK_NET -->|Outputs: SubnetIds & VpcId| STACK_DATA
    STACK_NET -->|Outputs: VpcId & SubnetIds| STACK_APP
    STACK_DATA -->|Outputs: DbEndpointArn & SecretArn| STACK_APP

    style STACK_NET fill:#0284c7,fill-opacity:0.08,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style STACK_DATA fill:#ef4444,fill-opacity:0.08,stroke:#ef4444,stroke-width:1.5px,color:inherit
    style STACK_APP fill:#10b981,fill-opacity:0.08,stroke:#10b981,stroke-width:2px,color:inherit
```

---

## 2. Safe Deployment Workflow: Change Sets & Drift Detection

Never execute in-place updates blindly. CloudFormation Change Sets allow developers to preview the blast radius before executing changes.

```mermaid
flowchart TD
    GIT["Developer Merges Pull Request"] --> SYNTH["CI/CD synthesizes CloudFormation Template"]
    SYNTH --> CS["<b>aws cloudformation create-change-set</b>"]

    subgraph AUDIT["Automated & Human Safety Gate"]
        DIFF{"Does Change Set contain Replacement of DB or S3?"}
        DIFF -->|Action: Modify / Add| SAFE["Preview Approved"]
        DIFF -->|Action: Replacement / Destroy on DB| ALARM["Pipeline Blocked! Manual Sign-off Required"]
    end

    CS --> AUDIT
    SAFE --> EXEC["<b>aws cloudformation execute-change-set</b>"]
    EXEC --> SUCCESS["Live Web Stack Updated with Zero Interruption"]

    style GIT fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style CS fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style AUDIT fill:#0284c7,fill-opacity:0.08,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style ALARM fill:#ef4444,fill-opacity:0.2,stroke:#ef4444,stroke-width:2px,color:inherit
    style SUCCESS fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
```

---

## 3. Web Stack CloudFormation Template Anatomy

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Production Web API Stack'

Parameters:
  Environment:
    Type: String
    Default: production
    AllowedValues: [staging, production]
  ContainerImage:
    Type: String
    Description: 'ECR Image URI with Git SHA tag'

Resources:
  # Application Load Balancer
  ApplicationLoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Scheme: internet-facing
      Subnets: !Split [',', !ImportValue 'NetworkStack-PublicSubnets']
      SecurityGroups: [!Ref AlbSecurityGroup]

  # ECS Web Service
  WebService:
    Type: AWS::ECS::Service
    Properties:
      Cluster: !ImportValue 'NetworkStack-EcsCluster'
      LaunchType: FARGATE
      DesiredCount: 3
      TaskDefinition: !Ref WebTaskDefinition
      LoadBalancers:
        - ContainerName: 'web-api'
          ContainerPort: 8080
          TargetGroupArn: !Ref WebTargetGroup

Outputs:
  AlbDnsName:
    Description: 'Public URL for Route 53'
    Value: !GetAtt ApplicationLoadBalancer.DNSName
    Export:
      Name: !Sub '${AWS::StackName}-AlbDns'
```
