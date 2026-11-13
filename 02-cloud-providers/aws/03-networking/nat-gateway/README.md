# Amazon VPC NAT Gateway

> **Focus**: Managed outbound-only internet connectivity for instances in private subnets.

---

## 1. Outbound Egress & Return Flow

```mermaid
flowchart LR
    subgraph PRIV["Private Subnet"]
        APP["EC2 Instance<br/>10.0.2.10 (No Public IP)"]
    end

    subgraph PUB["Public Subnet"]
        NAT["NAT Gateway<br/>Elastic IP: 52.12.34.56"]
    end

    IGW["Internet Gateway"]
    EXT["GitHub / External API"]

    APP -->|"1. Outbound Request (TCP SYN)"| NAT
    NAT -->|"2. SNAT (Replaces source IP with Elastic IP)"| IGW
    IGW -->|3. Route to Internet| EXT
    EXT -->|4. Response Packet| IGW
    IGW -->|5. Forward Return Stream| NAT
    NAT -->|6. Un-NAT back to 10.0.2.10| APP

    style PRIV fill:#f59e0b,fill-opacity:0.1,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style PUB fill:#10b981,fill-opacity:0.1,stroke:#10b981,stroke-width:1.5px,color:inherit
    style IGW fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
```

---

## 2. Multi-AZ High-Availability Best Practice

```mermaid
flowchart TD
    subgraph AZ_A["Availability Zone A"]
        NAT_A["NAT Gateway A (In Public A)"]
        PRIV_A["Private Subnet A"]
        PRIV_A -->|Route 0.0.0.0/0| NAT_A
    end

    subgraph AZ_B["Availability Zone B"]
        NAT_B["NAT Gateway B (In Public B)"]
        PRIV_B["Private Subnet B"]
        PRIV_B -->|Route 0.0.0.0/0| NAT_B
    end

    style AZ_A fill:#0284c7,fill-opacity:0.08,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style AZ_B fill:#10b981,fill-opacity:0.08,stroke:#10b981,stroke-width:1.5px,color:inherit
```

> [!CAUTION] Cross-AZ NAT Anti-Pattern
> Do **not** route traffic from Private Subnet B to NAT Gateway A in another AZ. You will incur cross-AZ data transfer fees and introduce a single point of failure.
