# 01 · AWS Compute Services

> **Module**: Compute  
> **Focus**: Workload execution models from virtual machines to serverless containers and event-driven functions.

---

## Compute Abstraction Spectrum

AWS offers a spectrum of compute abstractions balancing operational control against automated management.

```mermaid
flowchart LR
    EC2["<b>Amazon EC2</b><br/>IaaS / Raw VMs<br/>Full OS Control"] --> ECS["<b>Amazon ECS / EKS</b><br/>Container Orchestration<br/>Docker & Kubernetes"]
    ECS --> AR["<b>AWS App Runner</b><br/>PaaS Container Service<br/>Zero Infrastructure Ops"]
    AR --> LAMBDA["<b>AWS Lambda</b><br/>Serverless FaaS<br/>Pure Event Driven"]

    style EC2 fill:#3b82f6,fill-opacity:0.1,stroke:#3b82f6,stroke-width:2px,color:inherit
    style ECS fill:#0284c7,fill-opacity:0.1,stroke:#0284c7,stroke-width:2px,color:inherit
    style AR fill:#10b981,fill-opacity:0.1,stroke:#10b981,stroke-width:2px,color:inherit
    style LAMBDA fill:#f59e0b,fill-opacity:0.1,stroke:#f59e0b,stroke-width:2px,color:inherit
```

---

## Service Decision Matrix

```mermaid
flowchart TD
    Q1{"Do you need custom kernel, OS drivers, or legacy monolithic state?"}
    Q1 -->|Yes| EC2_RES["Use <b>Amazon EC2</b>"]
    Q1 -->|No| Q2{"Are you packaging apps into Docker containers?"}

    Q2 -->|No| Q3{"Is workload event-driven, short-lived (&lt;15m), or sporadic?"}
    Q3 -->|Yes| LAMBDA_RES["Use <b>AWS Lambda</b>"]
    Q3 -->|No| EC2_RES

    Q2 -->|Yes| Q4{"Need full Kubernetes ecosystem, CRDs & Helm?"}
    Q4 -->|Yes| EKS_RES["Use <b>Amazon EKS</b>"]
    Q4 -->|No| Q5{"Need simple HTTP web app without orchestration complexity?"}
    Q5 -->|Yes| AR_RES["Use <b>AWS App Runner</b>"]
    Q5 -->|No| ECS_RES["Use <b>Amazon ECS (Fargate)</b>"]

    style Q1 fill:#64748b,fill-opacity:0.15,stroke:#64748b,stroke-width:1.5px,color:inherit
    style Q2 fill:#64748b,fill-opacity:0.15,stroke:#64748b,stroke-width:1.5px,color:inherit
    style Q3 fill:#64748b,fill-opacity:0.15,stroke:#64748b,stroke-width:1.5px,color:inherit
    style Q4 fill:#64748b,fill-opacity:0.15,stroke:#64748b,stroke-width:1.5px,color:inherit
    style Q5 fill:#64748b,fill-opacity:0.15,stroke:#64748b,stroke-width:1.5px,color:inherit
    style EC2_RES fill:#3b82f6,fill-opacity:0.15,stroke:#3b82f6,stroke-width:1.5px,color:inherit
    style LAMBDA_RES fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style EKS_RES fill:#8b5cf6,fill-opacity:0.15,stroke:#8b5cf6,stroke-width:1.5px,color:inherit
    style AR_RES fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:1.5px,color:inherit
    style ECS_RES fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
```

---

## Compute Directory Index

- **[[ec2/README|EC2]]** — Elastic Compute Cloud instances, AMIs, storage attachment, and Auto Scaling.
- **[[lambda/README|Lambda]]** — Serverless Function-as-a-Service, lifecycle, execution environments, and concurrency.
- **[[ecs/README|ECS]]** — Elastic Container Service, Task Definitions, and Fargate serverless execution.
- **[[eks/README|EKS]]** — Elastic Kubernetes Service, managed control plane, and node groups.
- **[[app-runner/README|App Runner]]** — Fully managed container application runner from code or image.
