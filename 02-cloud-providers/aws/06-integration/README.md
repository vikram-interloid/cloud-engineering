# 06 · Application Integration & Messaging

> **Module**: Integration  
> **Focus**: Decoupled asynchronous architectures, queues, pub/sub, event routing, stateful orchestration, and APIs.

---

## AWS Messaging Patterns Spectrum

```mermaid
flowchart TD
    PATTERN{"Integration Pattern"}
    
    PATTERN -->|Point-to-Point Queue / Buffering / Rate Leveling| SQS["<b>Amazon SQS</b><br/>Pull-based, consumer polling, DLQ, at-least-once or FIFO"]
    PATTERN -->|Pub/Sub 1-to-Many Topic Fan-Out| SNS["<b>Amazon SNS</b><br/>Push-based, instant broadcast to thousands of endpoints"]
    PATTERN -->|Schema-based Event Bus with SaaS filtering| EB["<b>Amazon EventBridge</b><br/>Content-based filtering, AWS events & third-party SaaS"]
    PATTERN -->|Visual Multi-Step Workflow Orchestration| SF["<b>AWS Step Functions</b><br/>State machines, retry/catch, saga pattern, human-in-the-loop"]
    PATTERN -->|Synchronous HTTP/REST & WebSocket Entry Point| APIGW["<b>Amazon API Gateway</b><br/>Auth, rate throttling, payload validation, caching"]

    style PATTERN fill:#64748b,fill-opacity:0.15,stroke:#64748b,stroke-width:1.5px,color:inherit
    style SQS fill:#3b82f6,fill-opacity:0.15,stroke:#3b82f6,stroke-width:1.5px,color:inherit
    style SNS fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style EB fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style SF fill:#8b5cf6,fill-opacity:0.15,stroke:#8b5cf6,stroke-width:1.5px,color:inherit
    style APIGW fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
```

---

## Integration Directory Index

- **[[sqs/README|SQS]]** — Simple Queue Service, visibility timeout, and Dead-Letter Queues.
- **[[sns/README|SNS]]** — Simple Notification Service, pub/sub topics, and fan-out topologies.
- **[[eventbridge/README|EventBridge]]** — Universal event bus, content-based pattern matching, and registry.
- **[[step-functions/README|Step Functions]]** — Serverless workflow orchestration and Saga distributed transactions.
- **[[api-gateway/README|API Gateway]]** — REST, HTTP, and WebSocket APIs with built-in governance.
- **[[bedrock/README|Bedrock]]** — Foundation model APIs, generative AI guardrails, and model invocation.
- **[[ses/README|SES]]** — Simple Email Service, transactional email delivery, and domain verification.
