# Amazon VPC Internet Gateway (IGW)

> **Focus**: Managed horizontally scalable, highly available edge router connecting a VPC to the Internet.

---

## 1. 1:1 NAT and Bidirectional Routing

An Internet Gateway performs 1-to-1 Network Address Translation between private and public IP addresses with zero bandwidth bottlenecks.

```mermaid
flowchart LR
    subgraph VPC["VPC Subnet"]
        EC2["EC2 Instance<br/>Private: 10.0.1.25<br/>Public: 54.210.12.8"]
    end

    subgraph IGW_BOX["Internet Gateway (IGW)"]
        NAT_TABLE["1:1 NAT Engine<br/>Translates 10.0.1.25 &lt;--&gt; 54.210.12.8"]
    end

    INTERNET["Public Internet Client"]

    EC2 <==>|Private IP Packets| IGW_BOX
    IGW_BOX <==>|Public IP Packets| INTERNET

    style VPC fill:#0284c7,fill-opacity:0.08,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style IGW_BOX fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style INTERNET fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
```

---

## 2. Key Architectural Characteristics

- **Horizontally Scaled by AWS**: No availability constraints; scales automatically to hundreds of gigabits.
- **VPC Limit**: Exactly **one IGW** attached per VPC.
- **Stateless Router**: Simply performs address mapping and route translation.
