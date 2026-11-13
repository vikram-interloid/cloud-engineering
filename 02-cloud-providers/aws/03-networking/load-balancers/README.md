# AWS Elastic Load Balancing (ELB)

> **Focus**: Application Load Balancer (ALB), Network Load Balancer (NLB), and Gateway Load Balancer (GWLB).

---

## 1. ALB (Layer 7) vs. NLB (Layer 4)

```mermaid
flowchart TD
    CLIENT["Incoming Client Traffic"] --> DECIDE{"Protocol & Routing Needs"}
    
    DECIDE -->|HTTP/HTTPS, Path/Host routing, Microservices, TLS Termination| ALB["<b>Application Load Balancer (ALB)</b><br/>Layer 7 | HTTP/2 & gRPC | Path / Host Rules | WAF Integration"]
    DECIDE -->|TCP/UDP, Ultra-low latency, Static Elastic IP, Millions RPS| NLB["<b>Network Load Balancer (NLB)</b><br/>Layer 4 | Sub-millisecond | Preserves Source IP | High Throughput"]
    DECIDE -->|Third-party firewall appliance inspection| GWLB["<b>Gateway Load Balancer (GWLB)</b><br/>Layer 3 Gateway | GENEVE Tunneling | Inline Security Fleet"]

    style DECIDE fill:#64748b,fill-opacity:0.15,stroke:#64748b,stroke-width:1.5px,color:inherit
    style ALB fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style NLB fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style GWLB fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:2px,color:inherit
```

---

## 2. ALB Path-Based Routing to Target Groups

```mermaid
flowchart LR
    User["Client Request"] --> ALB["ALB Listener (:443)"]
    
    ALB -->|/api/users*| TG1["Target Group 1: Users Service (ECS Tasks)"]
    ALB -->|/api/orders*| TG2["Target Group 2: Orders Service (EC2 ASG)"]
    ALB -->|Default Rule| TG3["Target Group 3: Frontend Web (Lambda / S3)"]

    style ALB fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style TG1 fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:1.5px,color:inherit
    style TG2 fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style TG3 fill:#8b5cf6,fill-opacity:0.15,stroke:#8b5cf6,stroke-width:1.5px,color:inherit
```
