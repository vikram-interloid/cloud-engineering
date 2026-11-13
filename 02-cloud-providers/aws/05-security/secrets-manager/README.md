# AWS Secrets Manager

> **Focus**: Secure credential lifecycle, automated database password rotation, and audit compliance.

---

## 1. Automated Secret Rotation Architecture

```mermaid
sequenceDiagram
    autonumber
    participant SM as AWS Secrets Manager
    participant Lambda as Rotation Lambda Function
    participant RDS as Amazon RDS Database

    SM->>SM: Trigger Rotation Schedule (e.g. 30 days)
    SM->>Lambda: createSecret(VersionId)
    Lambda->>SM: Generate & Store Pending Secret
    SM->>Lambda: setSecret(VersionId)
    Lambda->>RDS: ALTER USER db_user IDENTIFIED BY 'new_password';
    SM->>Lambda: testSecret(VersionId)
    Lambda->>RDS: Verify connection with new password
    SM->>Lambda: finishSecret(VersionId)
    SM->>SM: Move AWSCURRENT stage label to new secret
```

---

## 2. Secrets Manager vs. SSM Parameter Store

```mermaid
flowchart TD
    STORE{"Secrets Management Need"}
    
    STORE -->|Automated rotation with RDS, cross-account sharing, automatic recovery| SM["<b>AWS Secrets Manager</b><br/>Cost: $0.40/secret/mo | Auto-Rotation built-in | Fine-grained auditing"]
    STORE -->|Configuration strings, license keys, plain hierarchical values| SSM["<b>SSM Parameter Store</b><br/>Cost: Free standard tier | Manual rotation | Native ECS/Lambda config"]

    style STORE fill:#64748b,fill-opacity:0.15,stroke:#64748b,stroke-width:1.5px,color:inherit
    style SM fill:#ef4444,fill-opacity:0.15,stroke:#ef4444,stroke-width:2px,color:inherit
    style SSM fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
```
