# Amazon SNS for Web Development

> **Focus**: Pub/Sub event broadcasting, fan-out architecture, transactional customer notifications (SMS/Email), and message filtering.

---

## 1. Web Application Event Fan-Out Architecture

When a business event occurs (e.g., `UserCheckoutCompleted`), the web application publishes a single message to an SNS topic. Downstream services consume independent copies via SQS queues without tight coupling.

```mermaid
flowchart TD
    CHECKOUT["Web App (Checkout Service)"] -->|1. Publish: OrderCreated| TOPIC["<b>Amazon SNS Topic</b><br/>(orders.fifo / orders)"]

    subgraph SUBSCRIBERS["Downstream Decoupled Web Services"]
        direction TB
        TOPIC -->|Fan-Out Copy| SQS_PAY["SQS: Payment Service Queue"]
        TOPIC -->|Fan-Out Copy| SQS_INV["SQS: Inventory Service Queue"]
        TOPIC -->|Fan-Out Copy| SQS_SHIP["SQS: Shipping Logistics Queue"]
        TOPIC -->|Subscription Filter: VIP Users| LAMBDA_NOTIF["Lambda: Real-Time Mobile Push Notification"]
    end

    SQS_PAY --> W_PAY["Payment Processing Workers"]
    SQS_INV --> W_INV["Warehouse Stock Reduction"]
    SQS_SHIP --> W_SHIP["Label Generation & Courier Dispatch"]

    style CHECKOUT fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style TOPIC fill:#f59e0b,fill-opacity:0.2,stroke:#f59e0b,stroke-width:2px,color:inherit
    style SUBSCRIBERS fill:#10b981,fill-opacity:0.06,stroke:#10b981,stroke-width:1.5px,color:inherit
```

---

## 2. Subscription Filter Policies (Smart Routing)

Instead of writing custom code in consumers to discard irrelevant messages, SNS subscription filter policies filter events at the SNS infrastructure tier.

```mermaid
flowchart LR
    MSG["Event Message:<br/>{ 'type': 'refund', 'amount': 450 }"] --> TOPIC["SNS Topic"]

    TOPIC -->|Filter: type == 'payment'| Q_PAY["Billing Queue (Skipped!)"]
    TOPIC -->|Filter: type == 'refund'| Q_REFUND["Refunds Team Queue (Delivered!)"]

    style TOPIC fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:2px,color:inherit
    style Q_REFUND fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style Q_PAY fill:#64748b,fill-opacity:0.1,stroke:#64748b,stroke-width:1px,color:inherit
```

---

## 3. Transactional Customer Communication (SMS & Email)

- **Direct SMS Dispatch**: Instant one-time passwords (OTP) and two-factor authentication codes delivered globally.
- **Delivery Status Logging**: Tracks whether SMS/Email carrier gateways successfully delivered notifications to users.
