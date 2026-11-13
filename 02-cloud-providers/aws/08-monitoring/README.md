# 08 · AWS Observability & Monitoring

> **Module**: Monitoring  
> **Focus**: The three pillars of observability: Metrics & Alarms, Centralized Logs, and Distributed Tracing.

---

## Observability Triad on AWS

```mermaid
flowchart TD
    subgraph OBSERVABILITY["AWS Observability Architecture"]
        direction LR
        
        subgraph METRICS["1. Metrics & Health"]
            CWM["<b>CloudWatch Metrics</b><br/>CPU, Latency, Error Rates, Alarms"]
        end

        subgraph LOGS["2. Centralized Logs"]
            CWL["<b>CloudWatch Logs</b><br/>Structured JSON, Logs Insights Queries"]
        end

        subgraph TRACES["3. Distributed Traces"]
            XRAY["<b>AWS X-Ray</b><br/>Microservice traces, Service Maps, Bottlenecks"]
        end
    end

    subgraph AUDIT["Governance & Compliance (Audit)"]
        CT["<b>AWS CloudTrail</b><br/>Who called what API at what time?"]
    end

    OBSERVABILITY --- AUDIT

    style METRICS fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style LOGS fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style TRACES fill:#8b5cf6,fill-opacity:0.15,stroke:#8b5cf6,stroke-width:2px,color:inherit
    style AUDIT fill:#ef4444,fill-opacity:0.15,stroke:#ef4444,stroke-width:2px,color:inherit
```

---

## Monitoring Directory Index

- **[[cloudwatch/README|CloudWatch]]** — Metrics, dynamic alarms, Logs Insights, and dashboards.
- **[[cloudtrail/README|CloudTrail]]** — API activity logging, governance auditing, and trail integrity.
- **[[x-ray/README|X-Ray]]** — Distributed transaction tracing, service latency maps, and subsegment analysis.
