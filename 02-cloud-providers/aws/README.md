# AWS Cloud Engineering Handbook

> **Path**: `02-cloud-providers/aws/`  
> **Target Audience**: Cloud Engineers, Solutions Architects, DevOps Practitioners  
> **Format**: Concise architectural summaries with high-density Mermaid visuals

---

## AWS Global Architecture Landscape

AWS infrastructure is organized into hierarchical compute, storage, networking, database, security, and integration layers connected by an ultra-low-latency private fiber backbone.

```mermaid
flowchart TD
    subgraph CLIENT["Clients & Edge Tier"]
        Users["Global Users / IoT / Mobile"]
        R53["Amazon Route 53 (DNS)"]
        CF["Amazon CloudFront (CDN Edge)"]
        WAF["AWS WAF (Layer 7 Defense)"]
        Users --> R53
        R53 --> CF
        CF --- WAF
    end

    subgraph INGRESS["Ingress & Networking Tier"]
        IGW["Internet Gateway"]
        ALB["Application Load Balancer"]
        NLB["Network Load Balancer"]
        CF --> IGW
        IGW --> ALB
        IGW --> NLB
    end

    subgraph COMPUTE["Compute & Application Tier"]
        direction TB
        subgraph SERVERS["IaaS / Containers"]
            EC2["Amazon EC2 (Instances)"]
            ECS["Amazon ECS (Containers)"]
            EKS["Amazon EKS (Kubernetes)"]
            AR["App Runner"]
        end
        subgraph SERVERLESS["Serverless / Event"]
            APIGW["API Gateway"]
            Lambda["AWS Lambda"]
            APIGW --> Lambda
        end
        ALB --> SERVERS
        ALB --> APIGW
    end

    subgraph DATA["Data & Persistence Tier"]
        subgraph RELATIONAL["Relational"]
            RDS["Amazon RDS"]
            Aurora["Amazon Aurora"]
        end
        subgraph NOSQL["NoSQL / In-Memory"]
            DDB["Amazon DynamoDB"]
            ElastiCache["ElastiCache (Redis)"]
        end
        subgraph OBJECT["Object & Analytics"]
            S3["Amazon S3"]
            RS["Amazon Redshift"]
        end
        SERVERS --> RDS
        SERVERS --> Aurora
        SERVERS --> ElastiCache
        Lambda --> DDB
        Lambda --> S3
        S3 --> RS
    end

    subgraph SEC_OBS["Security & Governance Cross-Cutting"]
        IAM["AWS IAM (Zero Trust)"]
        KMS["AWS KMS (Key Management)"]
        CW["CloudWatch (Logs/Metrics)"]
        CT["CloudTrail (Audit)"]
    end

    style CLIENT fill:#0284c7,fill-opacity:0.08,stroke:#0284c7,stroke-width:2px,color:inherit
    style INGRESS fill:#0d9488,fill-opacity:0.08,stroke:#0d9488,stroke-width:2px,color:inherit
    style COMPUTE fill:#f59e0b,fill-opacity:0.08,stroke:#f59e0b,stroke-width:2px,color:inherit
    style DATA fill:#6366f1,fill-opacity:0.08,stroke:#6366f1,stroke-width:2px,color:inherit
    style SEC_OBS fill:#dc2626,fill-opacity:0.08,stroke:#dc2626,stroke-width:2px,color:inherit
```

---

## Modular Curriculum Index

| Module | Title | Core Focus Areas |
| :--- | :--- | :--- |
| **[[00-introduction/README\|00 · Introduction]]** | Foundations | Global infrastructure, Multi-account AWS Organizations, Pricing |
| **[[01-compute/README\|01 · Compute]]** | Processing | EC2, Lambda, ECS, EKS, AWS App Runner |
| **[[02-storage/README\|02 · Storage]]** | Persistence | S3 Object, EBS Block, EFS File System, S3 Glacier Archive |
| **[[03-networking/README\|03 · Networking]]** | Connectivity | VPC, Subnets, Routing, IGW, NAT GW, ALB/NLB, CloudFront, Route 53 |
| **[[04-databases/README\|04 · Databases]]** | Data Engines | RDS, Aurora, DynamoDB NoSQL, ElastiCache In-Memory, Redshift OLAP |
| **[[05-security/README\|05 · Security]]** | Hardening & Auth | IAM RBAC/ABAC, KMS Encryption, Secrets Manager, Security Groups, WAF |
| **[[06-integration/README\|06 · Integration]]** | Decoupling | SQS Queues, SNS Pub/Sub, EventBridge Bus, Step Functions, API Gateway |
| **[[07-devops/README\|07 · DevOps & IaC]]** | Automation | AWS CDK, CloudFormation Stacks, CodePipeline CI/CD, CodeBuild |
| **[[08-monitoring/README\|08 · Monitoring]]** | Observability | CloudWatch Metrics/Logs, CloudTrail Auditing, X-Ray Distributed Tracing |
| **[[09-architecture-patterns/README\|09 · Architecture Patterns]]** | Blueprints | 3-Tier Enterprise, Serverless, Event-Driven, Microservices |
| **[[10-hands-on-labs/README\|10 · Hands-On Labs]]** | Implementation | Real-world guided implementations from scratch |
