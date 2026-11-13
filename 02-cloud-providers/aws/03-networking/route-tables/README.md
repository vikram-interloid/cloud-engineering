# VPC Route Tables

> **Focus**: Packet routing rules, longest-prefix-match algorithms, and gateway target associations.

---

## 1. Packet Routing Decision Engine

When a packet leaves an ENI, the VPC route table evaluates routes using **Longest Prefix Match** (most specific CIDR wins).

```mermaid
flowchart TD
    PACKET["Outgoing Packet: Destination 52.94.76.1"]
    
    subgraph RT["Route Table Evaluation Engine"]
        R1{"Matches 10.0.0.0/16? (/16 = 65,536 IPs)"}
        R2{"Matches 52.94.76.0/24? (/24 = 256 IPs)"}
        R3{"Matches 0.0.0.0/0? (/0 = Entire Internet)"}
    end

    PACKET --> RT
    R2 -->|Most Specific Match Wins!| TARGET["Target: NAT Gateway / IGW"]
    R3 -. Lower Priority Default .- FALLBACK["Target: 0.0.0.0/0"]

    style PACKET fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style RT fill:#0284c7,fill-opacity:0.08,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style TARGET fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
```

---

## 2. Typical Public vs Private Route Table Rules

```mermaid
flowchart LR
    subgraph PUB_RT["Public Route Table"]
        P1["10.0.0.0/16 -> local"]
        P2["0.0.0.0/0 -> igw-0123abcd"]
    end

    subgraph PRIV_RT["Private Route Table"]
        PR1["10.0.0.0/16 -> local"]
        PR2["0.0.0.0/0 -> nat-0987fedc"]
        PR3["pl-63a5400a (S3 Prefix) -> vpce-11223344"]
    end

    style PUB_RT fill:#10b981,fill-opacity:0.1,stroke:#10b981,stroke-width:1.5px,color:inherit
    style PRIV_RT fill:#f59e0b,fill-opacity:0.1,stroke:#f59e0b,stroke-width:1.5px,color:inherit
```
