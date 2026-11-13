# AWS Global Infrastructure

> **Focus**: Physical topology, failure domains, low-latency interconnection, and edge distribution.

---

## 1. Physical Infrastructure Hierarchy

AWS physical topology is designed to provide complete fault isolation across distinct failure domains.

```mermaid
flowchart TD
    subgraph GLOBAL["AWS Global Network (High-Capacity Private Fiber Backbone)"]
        direction TB

        subgraph REGION["AWS Region (e.g., us-east-1, eu-west-1)"]
            direction LR
            subgraph AZ1["Availability Zone A"]
                DC1["Data Center 1"]
                DC2["Data Center 2"]
            end
            subgraph AZ2["Availability Zone B"]
                DC3["Data Center 3"]
            end
            subgraph AZ3["Availability Zone C"]
                DC4["Data Center 4"]
                DC5["Data Center 5"]
            end
            AZ1 <== Ultra-Low Latency (<2ms) ==> AZ2
            AZ2 <== Dedicated Dark Fiber ==> AZ3
            AZ3 <== Redundant Transit Ring ==> AZ1
        end

        subgraph EDGE["Global Edge Network"]
            PoP1["CloudFront Edge Location"]
            PoP2["CloudFront Edge Location"]
            R53_Edge["Route 53 Anycast Node"]
        end
    end

    GLOBAL --> REGION
    GLOBAL --> EDGE

    style GLOBAL fill:#3b82f6,fill-opacity:0.05,stroke:#3b82f6,stroke-width:2px,color:inherit
    style REGION fill:#0284c7,fill-opacity:0.08,stroke:#0284c7,stroke-width:2px,color:inherit
    style AZ1 fill:#10b981,fill-opacity:0.12,stroke:#10b981,stroke-width:1.5px,color:inherit
    style AZ2 fill:#10b981,fill-opacity:0.12,stroke:#10b981,stroke-width:1.5px,color:inherit
    style AZ3 fill:#10b981,fill-opacity:0.12,stroke:#10b981,stroke-width:1.5px,color:inherit
    style EDGE fill:#f59e0b,fill-opacity:0.1,stroke:#f59e0b,stroke-width:1.5px,color:inherit
```

---

## 2. Infrastructure Comparison Matrix

| Component | Definition | Latency Characteristics | Fault Boundary |
| :--- | :--- | :--- | :--- |
| **Region** | Geographic area containing multiple isolated AZs | Independent geographic disaster domain | Regional disaster / legal jurisdiction |
| **Availability Zone (AZ)** | 1+ physical data centers with redundant power & cooling | `< 2 ms` round-trip between AZs | Flood, fire, local grid power failure |
| **Edge Location (PoP)** | Caching endpoint for CloudFront CDN & Route 53 | Nearest millisecond to internet end-user | Local ISP link or PoP hardware failure |
| **Local Zone** | Extension of AWS Region into metro areas | Single-digit ms to local users | Metro-level dependency on parent Region |
| **AWS Outposts** | Physical AWS-managed rack installed in on-prem DC | Sub-millisecond to on-prem workloads | Physical rack hardware and WAN uplink |

---

## 3. High-Availability Multi-AZ Design Rule

```mermaid
flowchart LR
    LB["ALB / Global Accelerator"]
    LB -->|Traffic 50%| AZ_A["AZ-a: Primary Instances / DB"]
    LB -->|Traffic 50%| AZ_B["AZ-b: Standby Instances / Replica"]
    AZ_A -.->|Synchronous Replication| AZ_B

    style LB fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style AZ_A fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:1.5px,color:inherit
    style AZ_B fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
```

- Always deploy stateless compute across **at least 2 AZs**.
- Rely on AWS managed replication for stateful data layers across AZs.
