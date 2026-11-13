# AWS IAM (Identity & Access Management)

> **Focus**: Zero Trust identity, IAM policy evaluation flowchart, and cross-account STS role assumption.

---

## 1. IAM Policy Evaluation Logic Flowchart

Every API call made to AWS evaluates policies using deterministic order-of-operations logic.

```mermaid
flowchart TD
    START["Incoming Request"] --> DEFAULT["Default: <b>DENY</b>"]
    
    DEFAULT --> SCP{"Restricted by SCP?"}
    SCP -->|Explicit Deny in SCP| DENIED["<b>ACCESS DENIED</b>"]
    SCP -->|Not Denied| PERM_BOUNDARY{"Restricted by Permission Boundary?"}

    PERM_BOUNDARY -->|Boundary Does Not Allow| DENIED
    PERM_BOUNDARY -->|Allowed| EXP_DENY{"Explicit Deny in any Policy?"}

    EXP_DENY -->|"Yes (Explicit Deny)"| DENIED
    EXP_DENY -->|No| EXP_ALLOW{"Explicit Allow in Identity/Resource Policy?"}

    EXP_ALLOW -->|"Yes (Explicit Allow)"| ALLOWED["<b>ACCESS GRANTED</b>"]
    EXP_ALLOW -->|"No (Implicit Deny)"| DENIED

    style START fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style DENIED fill:#ef4444,fill-opacity:0.2,stroke:#ef4444,stroke-width:2px,color:inherit
    style ALLOWED fill:#10b981,fill-opacity:0.2,stroke:#10b981,stroke-width:2px,color:inherit
```

---

## 2. STS Role Assumption (Cross-Account / Service Delegation)

```mermaid
sequenceDiagram
    autonumber
    actor Caller as EC2 / External Identity
    participant STS as AWS STS (AssumeRole API)
    participant Service as AWS S3 / DynamoDB

    Caller->>STS: AssumeRole (RoleArn, SessionName)
    STS->>STS: Validate Trust Policy
    STS-->>Caller: Temporary Credentials (AccessKey, SecretKey, SessionToken - 1h)
    Caller->>Service: GetObject (Using Temporary Credentials)
    Service-->>Caller: 200 OK
```
