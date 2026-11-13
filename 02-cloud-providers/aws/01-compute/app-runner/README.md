# AWS App Runner

> **Focus**: Fully managed containerized web applications and APIs with zero infrastructure orchestration.

---

## 1. App Runner Operating Model

Developers provide a source repository (GitHub) or container image (ECR). App Runner provisions load balancers, auto-scalers, TLS certificates, and health checks automatically.

```mermaid
flowchart LR
    subgraph SOURCE["Code / Artifact"]
        GH["GitHub Repository (Source Code)"]
        ECR["Amazon ECR (Container Image)"]
    end

    subgraph APPRUNNER["AWS App Runner Engine"]
        Build["Build / Deploy Pipeline"]
        Scale["Auto Scaling Service (Requests / Sec)"]
        TLS["Managed TLS & Custom Domains"]
        ALB_Managed["Built-in Load Balancer"]
        Build --> Scale --> TLS --> ALB_Managed
    end

    subgraph INSTANCES["App Runner Compute Instances"]
        I1["Container 1"]
        I2["Container 2"]
        I3["Container N"]
    end

    SOURCE --> APPRUNNER
    ALB_Managed --> INSTANCES
    Users["Internet Users"] -->|HTTPS| TLS

    style SOURCE fill:#3b82f6,fill-opacity:0.1,stroke:#3b82f6,stroke-width:1.5px,color:inherit
    style APPRUNNER fill:#10b981,fill-opacity:0.1,stroke:#10b981,stroke-width:2px,color:inherit
    style INSTANCES fill:#f59e0b,fill-opacity:0.1,stroke:#f59e0b,stroke-width:1.5px,color:inherit
```

---

## 2. App Runner vs ECS Fargate Comparison

| Feature | AWS App Runner | Amazon ECS on Fargate |
| :--- | :--- | :--- |
| **Primary Use Case** | Web applications, REST APIs | Complex microservices, background queues, batch |
| **Networking Setup** | Managed automatic endpoint & TLS | Requires manual VPC, Subnets, ALB, Target Groups |
| **Scaling Metric** | Concurrent incoming HTTP requests | CPU / Memory utilization or custom CloudWatch metrics |
| **Operational Overhead**| Near-Zero | Low-to-Medium |
