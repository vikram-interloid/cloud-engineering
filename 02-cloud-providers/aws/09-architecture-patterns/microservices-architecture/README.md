# Microservices Architecture Pattern

> **Focus**: Bounded contexts, independent scaling, service discovery, and database-per-service isolation.

---

## 1. Modern Microservices Architecture on AWS

```mermaid
flowchart TD
    CLIENT["API Consumers"] --> APIGW["Amazon API Gateway / ALB"]

    subgraph CLUSTER["Container Fleet (ECS / EKS on Fargate)"]
        subgraph USER_MS["Users Microservice"]
            U_SVC["Users Service (Task)"]
            U_DB[("Users DynamoDB")]
            U_SVC --> U_DB
        end

        subgraph ORDER_MS["Orders Microservice"]
            O_SVC["Orders Service (Task)"]
            O_DB[("Orders Aurora PostgreSQL")]
            O_SVC --> O_DB
        end

        subgraph INVENTORY_MS["Inventory Microservice"]
            I_SVC["Inventory Service (Task)"]
            I_CACHE[("Inventory ElastiCache")]
            I_SVC --> I_CACHE
        end
    end

    APIGW -->|/users| USER_MS
    APIGW -->|/orders| ORDER_MS
    APIGW -->|/inventory| INVENTORY_MS

    ORDER_MS -. Asynchronous Events .-> BUS["EventBridge / SQS"]
    BUS -. Sync Stock .-> INVENTORY_MS

    style CLIENT fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style APIGW fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style CLUSTER fill:#64748b,fill-opacity:0.06,stroke:#64748b,stroke-width:1.5px,color:inherit
    style USER_MS fill:#10b981,fill-opacity:0.1,stroke:#10b981,stroke-width:1px,color:inherit
    style ORDER_MS fill:#f59e0b,fill-opacity:0.1,stroke:#f59e0b,stroke-width:1px,color:inherit
    style INVENTORY_MS fill:#8b5cf6,fill-opacity:0.1,stroke:#8b5cf6,stroke-width:1px,color:inherit
```

---

## 2. Microservice Isolation Golden Rules

- **Database-per-Service**: Never allow Microservice A to query Microservice B's database directly.
- **API or Event Contracts**: Communication between domains must occur strictly via public REST APIs or asynchronous event schemas.
