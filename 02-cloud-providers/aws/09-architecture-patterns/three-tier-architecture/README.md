# Three-Tier Web Architecture Pattern

> **Focus**: Multi-AZ high availability, subnet tier isolation, and defense-in-depth networking.

---

## 1. Multi-AZ 3-Tier Architecture Blueprint

```mermaid
flowchart TD
    USERS["Internet Endpoints"] --> R53["Amazon Route 53"]
    R53 --> CF["CloudFront + AWS WAF"]

    subgraph VPC["AWS Virtual Private Cloud (Multi-AZ)"]
        IGW["Internet Gateway"]
        CF --> IGW

        subgraph PUBLIC_TIER["Tier 1: Web / Ingress (Public Subnets)"]
            ALB["Application Load Balancer (ALB)"]
            NAT_A["NAT Gateway AZ-a"]
            NAT_B["NAT Gateway AZ-b"]
            IGW --> ALB
        end

        subgraph APP_TIER["Tier 2: Business Logic (Private Subnets)"]
            subgraph ASG["EC2 Auto Scaling Group"]
                APP_A["App Server (AZ-a)"]
                APP_B["App Server (AZ-b)"]
            end
            ALB --> APP_A
            ALB --> APP_B
            APP_A -. Outbound updates .-> NAT_A
            APP_B -. Outbound updates .-> NAT_B
        end

        subgraph DATA_TIER["Tier 3: Persistence (Isolated Subnets)"]
            DB_PRI["Aurora Primary Writer (AZ-a)"]
            DB_SEC["Aurora Standby Reader (AZ-b)"]
            APP_A --> DB_PRI
            APP_B --> DB_PRI
            DB_PRI <== 6-Way Storage Replication ==> DB_SEC
        end
    end

    style VPC fill:#0284c7,fill-opacity:0.04,stroke:#0284c7,stroke-width:2px,color:inherit
    style PUBLIC_TIER fill:#10b981,fill-opacity:0.08,stroke:#10b981,stroke-width:1.5px,color:inherit
    style APP_TIER fill:#f59e0b,fill-opacity:0.08,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style DATA_TIER fill:#ef4444,fill-opacity:0.08,stroke:#ef4444,stroke-width:1.5px,color:inherit
```

---

## 2. Key Architecture Principles

- **Zero Public IPs in App/DB Tier**: Instances in Tiers 2 & 3 have private IP addresses only.
- **Strict Security Group Chaining**: Tier 2 SG allows ingress *only* from Tier 1 ALB SG; Tier 3 SG allows ingress *only* from Tier 2 App SG.
