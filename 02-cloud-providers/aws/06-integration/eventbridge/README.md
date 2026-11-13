# Amazon EventBridge (Serverless Event Bus)

> **Focus**: Content-based JSON event routing, SaaS partner integration, and scheduled rules.

---

## 1. Event Routing Pipeline

```mermaid
flowchart TD
    subgraph SOURCES["Event Sources"]
        AWS_EVT["AWS Services (S3, EC2 State Change)"]
        CUSTOM_EVT["Custom Apps (Microservices)"]
        SAAS_EVT["SaaS Partners (Shopify, Datadog, Auth0)"]
    end

    subgraph BUS["EventBridge Event Bus"]
        RULE1["Rule 1: Source = 'my.orders' AND total &gt; 100"]
        RULE2["Rule 2: Detail-Type = 'EC2 Instance State-change'"]
        RULE3["Rule 3: SaaS Webhook Notification"]
    end

    subgraph TARGETS["Target Consumers"]
        T1["AWS Lambda Function"]
        T2["Amazon SQS / SNS"]
        T3["Step Functions Workflow"]
        T4["API Gateway / EventBridge Pipes"]
    end

    SOURCES --> BUS
    RULE1 --> T1
    RULE1 --> T3
    RULE2 --> T2
    RULE3 --> T4

    style SOURCES fill:#0284c7,fill-opacity:0.08,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style BUS fill:#10b981,fill-opacity:0.1,stroke:#10b981,stroke-width:2px,color:inherit
    style TARGETS fill:#f59e0b,fill-opacity:0.08,stroke:#f59e0b,stroke-width:1.5px,color:inherit
```

---

## 2. EventBridge vs. SNS Comparison

| Feature | Amazon EventBridge | Amazon SNS |
| :--- | :--- | :--- |
| **Payload Filtering**| Inspects full JSON payload contents & arrays | Inspects message attributes only |
| **Latency** | Typical ~50ms | Ultra-low (sub-10ms) |
| **Third-Party SaaS** | Native Partner Event Source connectors | Requires custom webhook receiver |
| **Schema Registry** | Automatic schema discovery & code binding | No built-in schema registry |
