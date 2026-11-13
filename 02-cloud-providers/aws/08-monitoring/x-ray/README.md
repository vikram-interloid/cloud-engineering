# AWS X-Ray (Distributed Tracing)

> **Focus**: End-to-end request tracing across distributed microservices, service maps, and latency debugging.

---

## 1. Distributed Trace & Service Map

X-Ray propagates a trace header (`X-Amzn-Trace-Id`) across service boundaries to construct a unified latency graph.

```mermaid
flowchart LR
    CLIENT["Client Browser"] -->|Trace ID: 1-5e...| APIGW["Amazon API Gateway"]
    APIGW -->|Propagate Trace| LAMBDA["Order Lambda Handler"]
    
    LAMBDA -->|Trace Subsegment 1| DDB["Amazon DynamoDB (12ms)"]
    LAMBDA -->|Trace Subsegment 2| SQS["Amazon SQS Queue (4ms)"]
    LAMBDA -->|Trace Subsegment 3| EXT["External Payment Gateway (450ms Latency Bottleneck!)"]

    style CLIENT fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style APIGW fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style LAMBDA fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style EXT fill:#ef4444,fill-opacity:0.2,stroke:#ef4444,stroke-width:2px,color:inherit
```

---

## 2. Trace Anatomy: Segments vs. Subsegments

- **Segment**: Represents work done by a compute resource (e.g., the Lambda function invocation or EC2 web server).
- **Subsegment**: Breaks down internal downstream operations within that segment (e.g., an outbound SQL query, HTTP call, or S3 read).
