# Lab 02 · Deploy an EC2 Web Server

> **Objective**: Launch an Amazon Linux 2023 EC2 instance with automated User Data bootstrapping, Security Group rules, and public HTTP verification.

---

## Target Architecture

```mermaid
flowchart LR
    BROWSER["Web Browser<br/>(Port 80)"] --> SG{"Security Group<br/>Inbound: 80 from 0.0.0.0/0<br/>Inbound: 22 from MyIP"}
    SG --> EC2["EC2 Instance (Amazon Linux 2023)<br/>Runs NGINX via User Data"]
    EC2 --> EBS["EBS Root Volume (gp3 Encrypted)"]

    style BROWSER fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style SG fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style EC2 fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style EBS fill:#3b82f6,fill-opacity:0.15,stroke:#3b82f6,stroke-width:1.5px,color:inherit
```

---

## User Data Boot Script (`userdata.sh`)

```bash
#!/bin/bash
dnf update -y
dnf install -y nginx
systemctl start nginx
systemctl enable nginx
echo "<h1>Deployed via AWS EC2 User Data</h1>" > /usr/share/nginx/html/index.html
```

---

## CLI Deployment Command

```bash
aws ec2 run-instances \
  --image-id ami-0c101f26f147fa7fd \
  --instance-type t3.micro \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0 \
  --user-data file://userdata.sh \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=web-server-demo}]'
```
