# Amazon VPC Subnets

> **Focus**: Subnet segmentation strategies: Public, Private, and Isolated tiers across AZs.

---

## 1. 3-Tier Subnet Segmentation Pattern

```mermaid
flowchart TD
    subgraph SUBNETS["Multi-AZ Subnet Segmentation"]
        direction TB

        subgraph TIER1["1. Public Subnet Tier (Internet-Facing)"]
            PUB_A["Public Subnet AZ-a<br/>Route: 0.0.0.0/0 -> IGW"]
            PUB_B["Public Subnet AZ-b<br/>Route: 0.0.0.0/0 -> IGW"]
            ALB["ALB & NAT Gateways"]
        end

        subgraph TIER2["2. Private Subnet Tier (Internal Applications)"]
            PRIV_A["Private Subnet AZ-a<br/>Route: 0.0.0.0/0 -> NAT-GW-a"]
            PRIV_B["Private Subnet AZ-b<br/>Route: 0.0.0.0/0 -> NAT-GW-b"]
            APPS["EC2 / ECS / EKS Workloads"]
        end

        subgraph TIER3["3. Isolated Subnet Tier (Data Stores)"]
            ISO_A["Isolated Subnet AZ-a<br/>Route: Local Only (No Internet)"]
            ISO_B["Isolated Subnet AZ-b<br/>Route: Local Only (No Internet)"]
            DBS["RDS / Aurora / ElastiCache"]
        end

        TIER1 --> TIER2 --> TIER3
    end

    style TIER1 fill:#10b981,fill-opacity:0.1,stroke:#10b981,stroke-width:1.5px,color:inherit
    style TIER2 fill:#f59e0b,fill-opacity:0.1,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style TIER3 fill:#ef4444,fill-opacity:0.1,stroke:#ef4444,stroke-width:1.5px,color:inherit
```

---

## 2. Subnet Classification Matrix

| Subnet Type | Route Table Target for `0.0.0.0/0` | Public IP Auto-Assign | Typical Resources |
| :--- | :--- | :--- | :--- |
| **Public** | `igw-xxxxxxxx` (Internet Gateway) | Yes (Optional) | ALB, NLB, NAT Gateway, Bastion |
| **Private** | `nat-xxxxxxxx` (NAT Gateway) | No | Microservices, Backend APIs, Workers |
| **Isolated** | None (Local VPC route only) | No | RDS Databases, Redis Caches, KMS HSMs |
