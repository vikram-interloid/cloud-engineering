# 09 · Production Architecture Patterns

> **Module**: Architecture Patterns  
> **Focus**: Reference architectures, Well-Architected Framework principles, and trade-off analyses.

---

## Pattern Comparison Matrix

```mermaid
flowchart TD
    GOAL{"Application Architecture Objective"}
    
    GOAL -->|Traditional enterprise, stateful backend, strict network tiers| T3["<b>Three-Tier Web Architecture</b><br/>Multi-AZ VPC, ALB, Auto-scaling EC2, RDS Aurora"]
    GOAL -->|Zero idle cost, rapid time-to-market, event-based scale| SRV["<b>Serverless Architecture</b><br/>CloudFront, S3 SPA, API Gateway, Lambda, DynamoDB"]
    GOAL -->|High decouple, asynchronous processing, spike absorption| EDA["<b>Event-Driven Architecture</b><br/>EventBridge, SNS fan-out, SQS buffers, Step Functions"]
    GOAL -->|Domain-driven teams, independent release cycles, containers| MIC["<b>Microservices Architecture</b><br/>Amazon EKS / ECS Fargate, Service Connect, PrivateLink"]

    style GOAL fill:#64748b,fill-opacity:0.15,stroke:#64748b,stroke-width:1.5px,color:inherit
    style T3 fill:#3b82f6,fill-opacity:0.15,stroke:#3b82f6,stroke-width:1.5px,color:inherit
    style SRV fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style EDA fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style MIC fill:#8b5cf6,fill-opacity:0.15,stroke:#8b5cf6,stroke-width:1.5px,color:inherit
```

---

## Architecture Patterns Directory Index

- **[[three-tier-architecture/README|Three-Tier Architecture]]** — Classic enterprise Web-App-Database multi-AZ design.
- **[[serverless-architecture/README|Serverless Architecture]]** — 100% serverless web and API pattern.
- **[[event-driven-architecture/README|Event-Driven Architecture]]** — Asynchronous choreography and event-sourced workflows.
- **[[microservices-architecture/README|Microservices Architecture]]** — Containerized domain microservices with decentralized data.
