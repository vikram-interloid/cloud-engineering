# 10 · AWS Hands-On Labs

> **Module**: Practical Labs  
> **Focus**: Step-by-step implementation playbooks with commands and architecture blueprints.

---

## Guided Labs Learning Path

```mermaid
flowchart LR
    LAB1["<b>Lab 01</b><br/>Create Custom VPC"] --> LAB2["<b>Lab 02</b><br/>Deploy Secure EC2"]
    LAB2 --> LAB3["<b>Lab 03</b><br/>Serverless CRUD API"]
    LAB3 --> LAB4["<b>Lab 04</b><br/>Container App on ECS"]

    style LAB1 fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style LAB2 fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style LAB3 fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:2px,color:inherit
    style LAB4 fill:#8b5cf6,fill-opacity:0.15,stroke:#8b5cf6,stroke-width:2px,color:inherit
```

---

## Labs Directory Index

- **[[create-vpc/README|Create Custom VPC]]** — Multi-AZ VPC with Public/Private subnets, IGW, and NAT.
- **[[deploy-ec2/README|Deploy EC2 Web Server]]** — Launch Linux instance with automated User Data and Security Groups.
- **[[deploy-serverless-api/README|Deploy Serverless API]]** — Build a scalable CRUD API with API Gateway, Lambda, and DynamoDB.
- **[[deploy-container-application/README|Deploy Container App]]** — Package Docker app, push to ECR, and launch on ECS Fargate behind ALB.
