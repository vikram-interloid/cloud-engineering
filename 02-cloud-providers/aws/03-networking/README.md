# 03 · AWS Networking & Content Delivery

> **Module**: Networking  
> **Focus**: Software-Defined Networking (SDN), VPC topologies, secure routing, traffic distribution, CDN, and DNS.

---

## VPC End-to-End Traffic Blueprint

```mermaid
flowchart TD
    INTERNET["Public Internet Client"]
    R53["Amazon Route 53 (DNS)"]
    CF["Amazon CloudFront (Edge Caching)"]
    
    INTERNET -->|DNS Query| R53
    INTERNET -->|HTTPS Request| CF

    subgraph VPC["Amazon VPC (10.0.0.0/16)"]
        IGW["Internet Gateway"]
        CF --> IGW

        subgraph PUBLIC_TIER["Public Subnet (10.0.1.0/24)"]
            ALB["Application Load Balancer"]
            NAT["NAT Gateway"]
            IGW <--> ALB
            IGW <--> NAT
        end

        subgraph PRIVATE_TIER["Private Application Subnet (10.0.2.0/24)"]
            APP1["EC2 / ECS App Server"]
            ALB --> APP1
            APP1 -->|Outbound Updates| NAT
        end

        subgraph ISOLATED_TIER["Isolated Database Subnet (10.0.3.0/24)"]
            DB1["RDS Aurora Database"]
            APP1 --> DB1
        end
    end

    style VPC fill:#0284c7,fill-opacity:0.05,stroke:#0284c7,stroke-width:2px,color:inherit
    style PUBLIC_TIER fill:#10b981,fill-opacity:0.1,stroke:#10b981,stroke-width:1.5px,color:inherit
    style PRIVATE_TIER fill:#f59e0b,fill-opacity:0.1,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style ISOLATED_TIER fill:#ef4444,fill-opacity:0.1,stroke:#ef4444,stroke-width:1.5px,color:inherit
```

---

## Networking Directory Index

- **[[vpc/README|VPC]]** — Virtual Private Cloud isolation, CIDR design, and VPC Peering / PrivateLink.
- **[[subnets/README|Subnets]]** — Public, Private, and Isolated tier segmentation.
- **[[route-tables/README|Route Tables]]** — Packet destination lookup and longest-prefix-match rules.
- **[[internet-gateway/README|Internet Gateway]]** — Bidirectional 1:1 NAT public ingress and egress.
- **[[nat-gateway/README|NAT Gateway]]** — Outbound-only internet egress for private workloads.
- **[[load-balancers/README|Load Balancers]]** — ALB (Layer 7), NLB (Layer 4), and Gateway Load Balancer.
- **[[cloudfront/README|CloudFront]]** — Global CDN, edge caching, and Origin Shield.
- **[[route53/README|Route 53]]** — 100% SLA DNS resolution and active-passive failover routing.
