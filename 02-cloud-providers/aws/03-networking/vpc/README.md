# Amazon VPC (Virtual Private Cloud)

> **Focus**: Logically isolated software-defined virtual networks, CIDR allocation, and connectivity topologies.

---

## 1. VPC CIDR & IP Addressing

```mermaid
flowchart TD
    VPC["<b>VPC CIDR</b>: 10.0.0.0/16 (65,536 Total IPs)"]

    VPC --> AZ1_PUB["Subnet AZ-a Public: 10.0.1.0/24 (251 usable IPs)"]
    VPC --> AZ1_PRIV["Subnet AZ-a Private: 10.0.2.0/24 (251 usable IPs)"]
    VPC --> AZ2_PUB["Subnet AZ-b Public: 10.0.11.0/24 (251 usable IPs)"]
    VPC --> AZ2_PRIV["Subnet AZ-b Private: 10.0.12.0/24 (251 usable IPs)"]

    style VPC fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style AZ1_PUB fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:1.5px,color:inherit
    style AZ1_PRIV fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style AZ2_PUB fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:1.5px,color:inherit
    style AZ2_PRIV fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
```

> [!NOTE] AWS Reserved IP Addresses (First 4 and Last 1)
> In every subnet (e.g., `10.0.1.0/24`), 5 IPs are reserved:
> - `10.0.1.0`: Network address
> - `10.0.1.1`: VPC router
> - `10.0.1.2`: Amazon DNS server
> - `10.0.1.3`: Future AWS reservation
> - `10.0.1.255`: Network broadcast address

---

## 2. VPC Peering vs. AWS Transit Gateway

```mermaid
flowchart LR
    subgraph PEERING["VPC Peering (Mesh / Non-transitive)"]
        direction TB
        V1["VPC A"] <--> V2["VPC B"]
        V2 <--> V3["VPC C"]
        V1 -. No Direct Transit .- V3
    end

    subgraph TGW["AWS Transit Gateway (Hub & Spoke / Transitive)"]
        direction TB
        HUB(("Transit Gateway Hub"))
        VA["VPC A"] <--> HUB
        VB["VPC B"] <--> HUB
        VC["VPC C"] <--> HUB
        VPN["On-Premises IPsec / DX"] <--> HUB
    end

    style PEERING fill:#0284c7,fill-opacity:0.08,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style TGW fill:#10b981,fill-opacity:0.08,stroke:#10b981,stroke-width:1.5px,color:inherit
```
