# Amazon EFS (Elastic File System)

> **Focus**: Serverless, elastic, POSIX-compliant shared network file storage (NFSv4).

---

## 1. Multi-AZ Shared File System Architecture

Amazon EFS provides concurrent read/write file access to thousands of instances across multiple Availability Zones simultaneously.

```mermaid
flowchart TD
    subgraph VPC["Virtual Private Cloud (VPC)"]
        subgraph AZ_A["Availability Zone A"]
            EC2_A["EC2 Instance"]
            ECS_A["ECS Task"]
            MT_A["EFS Mount Target A"]
            EC2_A --> MT_A
            ECS_A --> MT_A
        end

        subgraph AZ_B["Availability Zone B"]
            EC2_B["EC2 Instance"]
            LAMBDA_B["AWS Lambda"]
            MT_B["EFS Mount Target B"]
            EC2_B --> MT_B
            LAMBDA_B --> MT_B
        end
    end

    subgraph EFS_SERVICE["Amazon EFS Storage Cluster (Multi-AZ Replicated)"]
        EFS_STORAGE["POSIX File System<br/>(Auto scales up & down automatically)"]
    end

    MT_A <== NFSv4.1 ==> EFS_STORAGE
    MT_B <== NFSv4.1 ==> EFS_STORAGE

    style VPC fill:#0284c7,fill-opacity:0.06,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style AZ_A fill:#10b981,fill-opacity:0.1,stroke:#10b981,stroke-width:1px,color:inherit
    style AZ_B fill:#f59e0b,fill-opacity:0.1,stroke:#f59e0b,stroke-width:1px,color:inherit
    style EFS_SERVICE fill:#8b5cf6,fill-opacity:0.1,stroke:#8b5cf6,stroke-width:2px,color:inherit
```

---

## 2. EFS Performance & Storage Tiers

- **Storage Classes**: Standard (Active) & Infrequent Access (EFS-IA) with automated lifecycle management.
- **Throughput Modes**:
  - **Elastic**: Automatically scales throughput to match workload demand (recommended).
  - **Provisioned**: Dedicated throughput regardless of stored volume size.
