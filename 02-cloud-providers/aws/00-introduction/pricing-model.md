# AWS Pricing & Financial Models

> **Focus**: Compute purchasing options, commitment models, and cost optimization decision logic.

---

## 1. Compute Purchasing Models

AWS provides 4 primary purchase vehicles balancing commitment, flexibility, and cost.

```mermaid
flowchart TD
    START{"Compute Workload Profile"}
    
    START -->|Short-term, unpredictable, spikey| OD["<b>On-Demand</b><br/>No commitment, maximum flexibility, baseline price"]
    START -->|Fault-tolerant, batch, stateless, containerized| SPOT["<b>Spot Instances</b><br/>Up to 90% discount, 2-min termination notice"]
    START -->|Predictable steady-state across 1 or 3 years| SP["<b>Savings Plans</b><br/>Up to 72% discount, $/hr spend commitment"]
    START -->|Predictable steady-state database/cache instances| RI["<b>Reserved Instances (RI)</b><br/>Standard or Convertible reserved capacity"]

    style START fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style OD fill:#ef4444,fill-opacity:0.15,stroke:#ef4444,stroke-width:1.5px,color:inherit
    style SPOT fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:1.5px,color:inherit
    style SP fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style RI fill:#8b5cf6,fill-opacity:0.15,stroke:#8b5cf6,stroke-width:1.5px,color:inherit
```

---

## 2. Cost vs. Flexibility Spectrum

```mermaid
flowchart LR
    Spot["Spot Instances<br/>(Lowest Cost / Lowest SLA)"] --- SP_EC2["Compute Savings Plans"] --- RI_Std["Standard RIs"] --- OD["On-Demand<br/>(Highest Cost / Highest Flexibility)"]

    style Spot fill:#10b981,fill-opacity:0.2,stroke:#10b981,stroke-width:2px,color:inherit
    style SP_EC2 fill:#06b6d4,fill-opacity:0.2,stroke:#06b6d4,stroke-width:2px,color:inherit
    style RI_Std fill:#f59e0b,fill-opacity:0.2,stroke:#f59e0b,stroke-width:2px,color:inherit
    style OD fill:#ef4444,fill-opacity:0.2,stroke:#ef4444,stroke-width:2px,color:inherit
```

---

## 3. Financial Optimization Matrix

| Tier | Commitment | Discount | Ideal Workload |
| :--- | :--- | :--- | :--- |
| **On-Demand** | None (per-second billing) | 0% | Dev/testing, unpredictable traffic, first 30 days |
| **Compute Savings Plans** | 1 or 3 Years ($/hr commit) | Up to 66% | General compute (EC2, Fargate, Lambda) |
| **EC2 Instance Savings Plans**| 1 or 3 Years (specific family)| Up to 72% | Static instance family in a specific region |
| **Spot Instances** | None (Spare capacity bidding)| Up to 90% | Stateless microservices, CI builds, big data / ML |
