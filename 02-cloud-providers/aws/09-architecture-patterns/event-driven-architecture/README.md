# Event-Driven Architecture (EDA) Pattern

> **Focus**: Asynchronous event choreography, decoupled microservices, and resilient spike leveling.

---

## 1. Choreographed E-Commerce Event Workflow

```mermaid
flowchart LR
    ORDER["<b>Order Service</b><br/>Publishes: OrderPlaced"] --> BUS["<b>Amazon EventBridge Bus</b>"]

    subgraph CONSUMERS["Decoupled Domain Consumers"]
        BUS -->|Rule 1| PAY["<b>Payment Service</b><br/>(Lambda + Stripe)"]
        BUS -->|Rule 2| INV["<b>Inventory Service</b><br/>(SQS + ECS Task)"]
        BUS -->|Rule 3| NOTIF["<b>Notification Service</b><br/>(SNS + SES Email)"]
        BUS -->|Rule 4| ARCHIVE["<b>Event Store</b><br/>(Firehose -> S3 Data Lake)"]
    end

    PAY -. Emits: PaymentSuccess .-> BUS
    INV -. Emits: InventoryReserved .-> BUS

    style ORDER fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style BUS fill:#f59e0b,fill-opacity:0.2,stroke:#f59e0b,stroke-width:2px,color:inherit
    style CONSUMERS fill:#10b981,fill-opacity:0.06,stroke:#10b981,stroke-width:1.5px,color:inherit
```

---

## 2. Choreography vs. Orchestration

```mermaid
flowchart TD
    subgraph CHOREO["Choreography (Event-Driven)"]
        C_A["Service A"] -->|Event: Done| C_BUS((Event Bus))
        C_BUS --> C_B["Service B"]
        C_BUS --> C_C["Service C"]
    end

    subgraph ORCH["Orchestration (Step Functions)"]
        SF_ENGINE["Central State Machine Controller"]
        SF_ENGINE -->|1. Call| S_A["Task A"]
        SF_ENGINE -->|2. Evaluate Choice & Call| S_B["Task B"]
    end

    style CHOREO fill:#10b981,fill-opacity:0.1,stroke:#10b981,stroke-width:1.5px,color:inherit
    style ORCH fill:#8b5cf6,fill-opacity:0.1,stroke:#8b5cf6,stroke-width:1.5px,color:inherit
```
