# AWS CloudTrail

> **Focus**: Comprehensive API auditing, security forensic investigation, and compliance tracking.

---

## 1. CloudTrail Event Delivery Pipeline

CloudTrail records every API call made in the AWS account (Console, CLI, SDK, and service-to-service).

```mermaid
flowchart TD
    ACTOR["User / Service API Action<br/>(e.g., RunInstances, DeleteBucket)"] --> CT["<b>AWS CloudTrail Engine</b><br/>Captures Caller Identity, Timestamp, IP, Request Parameters"]

    CT --> S3["<b>Encrypted S3 Audit Bucket</b><br/>Permanent storage (Immutable Log Archive)"]
    CT --> CWL["<b>CloudWatch Logs</b><br/>Real-time stream for security anomaly detection"]
    
    CWL --> METRIC["Metric Filter: 'UnauthorizedAccess'"]
    METRIC --> ALERT["SNS Security Alert (SecOps)"]

    style ACTOR fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style CT fill:#ef4444,fill-opacity:0.15,stroke:#ef4444,stroke-width:2px,color:inherit
    style S3 fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:1.5px,color:inherit
    style CWL fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
```

---

## 2. Event Types Reference

| Event Category | Scope | Examples | Default Enabled |
| :--- | :--- | :--- | :--- |
| **Management Events** | Control plane actions on resources | `CreateBucket`, `TerminateInstances`, `AttachPolicy` | Yes (90-day free event history) |
| **Data Events** | High-volume data plane actions | `S3:GetObject`, `DynamoDB:PutItem`, `Lambda:Invoke` | No (Opt-in per resource) |
| **Insights Events** | Anomaly detection for unusual API call spikes | Sudden spike in IAM write calls | No (Opt-in) |
