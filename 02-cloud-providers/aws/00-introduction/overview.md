# AWS Overview & Platform Architecture

> **Focus**: The core operating model of Amazon Web Services (AWS), unified API planes, and security accountability.

---

## 1. Unified Control Plane vs Data Plane

Every AWS resource is provisioned, mutated, and inspected through a unified, authenticated REST API control plane.

```mermaid
flowchart TD
    subgraph INTERFACES["Management Interfaces"]
        CLI["AWS CLI"]
        Console["AWS Management Console"]
        SDK["AWS SDKs / CDK"]
        IaC["Terraform / Pulumi"]
    end

    subgraph CONTROL["AWS Control Plane (API Gateway & IAM)"]
        SigV4["SigV4 Authentication"]
        IAM_Check["IAM Authorization Engine"]
        APIS["AWS Service APIs (ec2.amazonaws.com, s3.amazonaws.com)"]
        SigV4 --> IAM_Check --> APIS
    end

    subgraph DATA_PLANE["AWS Data Plane (Execution Layer)"]
        Hypervisors["Nitro System / Hypervisors"]
        EBS_Fabric["EBS Storage Fabric"]
        VPC_Fabric["Nitro Network Enclaves"]
    end

    INTERFACES -->|Signed HTTPS Request| SigV4
    APIS -->|Command & Control| DATA_PLANE

    style INTERFACES fill:#0284c7,fill-opacity:0.1,stroke:#0284c7,stroke-width:2px,color:inherit
    style CONTROL fill:#f59e0b,fill-opacity:0.1,stroke:#f59e0b,stroke-width:2px,color:inherit
    style DATA_PLANE fill:#10b981,fill-opacity:0.1,stroke:#10b981,stroke-width:2px,color:inherit
```

---

## 2. AWS Shared Responsibility Model

Security and compliance are a shared effort between AWS and the customer.

```mermaid
flowchart TD
    subgraph CUSTOMER["CUSTOMER RESPONSIBILITY ('Security IN the Cloud')"]
        C1["Customer Data & Classification"]
        C2["IAM Access Management & MFA"]
        C3["OS Patching & Application Firewalls"]
        C4["Network Configuration & Security Groups"]
        C5["Client & Server Data Encryption"]
    end

    subgraph AWS["AWS RESPONSIBILITY ('Security OF the Cloud')"]
        A1["Regions, AZs & Data Center Physical Security"]
        A2["Nitro Hardware, Host OS & Hypervisor"]
        A3["Global Network Infrastructure & Fiber Lines"]
        A4["Managed Service Foundations (S3, DynamoDB, RDS engines)"]
    end

    CUSTOMER ---|Demarcation Line| AWS

    style CUSTOMER fill:#3b82f6,fill-opacity:0.1,stroke:#3b82f6,stroke-width:2px,color:inherit
    style AWS fill:#ef4444,fill-opacity:0.1,stroke:#ef4444,stroke-width:2px,color:inherit
```

---

## 3. Core Architectural Pillars

- **Zero-Trust by Default**: No resource has permissions until explicitly granted by IAM policies.
- **Microsecond Nitro Hypervisor**: Custom ASIC chips offload storage, network, and security processing from host CPUs.
- **Multi-Tenant Isolation**: Cryptographic network isolation inside the Amazon VPC network fabric.
