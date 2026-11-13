# Amazon CloudWatch

> **Focus**: Operational metrics, high-resolution telemetry, log stream analysis, and automated remediation.

---

## 1. CloudWatch Metrics & Alarm Remediation Flow

```mermaid
flowchart LR
    APP["Workload (EC2 / ECS / Lambda)"] -->|PutMetricData| CWM["<b>CloudWatch Metrics</b>"]
    
    CWM --> ALARM{"Alarm Threshold<br/>(e.g., ErrorRate &gt; 5% for 3 mins)"}
    
    ALARM -->|Trigger ALARM State| ACTION1["SNS Notification (PagerDuty / Slack)"]
    ALARM -->|Trigger Auto-Scaling| ACTION2["EC2 Auto Scaling / ECS Scale-Out"]
    ALARM -->|Automated Recovery| ACTION3["EC2 Instance Reboot / Auto-Recovery"]

    style APP fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style CWM fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style ALARM fill:#ef4444,fill-opacity:0.15,stroke:#ef4444,stroke-width:2px,color:inherit
    style ACTION1 fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style ACTION2 fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:1.5px,color:inherit
```

---

## 2. CloudWatch Logs Insights Quick Syntax

```mermaid
flowchart TD
    LOGS["Log Group: /aws/lambda/orders-api"] --> QUERY["Logs Insights Query Engine"]
    QUERY --> RESULTS["Fast Aggregated Results"]

    style LOGS fill:#0284c7,fill-opacity:0.1,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style QUERY fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
```

```sql
fields @timestamp, @message
| filter @message like /ERROR/
| stats count(*) as errorCount by bin(5m)
| sort errorCount desc
```
