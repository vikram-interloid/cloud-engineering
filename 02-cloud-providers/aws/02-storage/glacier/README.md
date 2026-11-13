# Amazon S3 Glacier

> **Focus**: Cold data archiving, compliance retention, retrieval tiers, and Vault Lock policies.

---

## 1. Glacier Retrieval Tiers & Latency

```mermaid
flowchart TD
    ARCHIVE["Archived Data Request"]
    
    ARCHIVE -->|Milliseconds| INSTANT["<b>Glacier Instant Retrieval</b><br/>Cost: Low | Latency: Milliseconds | Access: Once a quarter"]
    ARCHIVE -->|Minutes to Hours| FLEX["<b>Glacier Flexible Retrieval</b><br/>Cost: Lower | Latency: 1-5 min (Expedited), 3-5 hrs (Standard)"]
    ARCHIVE -->|Hours| DEEP["<b>Glacier Deep Archive</b><br/>Cost: Lowest ($0.00099/GB) | Latency: 12-48 hours"]

    style ARCHIVE fill:#64748b,fill-opacity:0.15,stroke:#64748b,stroke-width:1.5px,color:inherit
    style INSTANT fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:1.5px,color:inherit
    style FLEX fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style DEEP fill:#8b5cf6,fill-opacity:0.15,stroke:#8b5cf6,stroke-width:1.5px,color:inherit
```

---

## 2. S3 Glacier Vault Lock (WORM Compliance)

```mermaid
flowchart LR
    Policy["Vault Lock Policy JSON<br/>(Write Once, Read Many)"] --> Initiate["InitiateLock API<br/>(24-Hour Testing Window)"]
    Initiate --> Complete["CompleteLock API<br/>(Policy Locked Permanently)"]
    Complete --> Enforce["Immutable WORM Enforcement<br/>(Cannot be deleted, even by AWS Root User)"]

    style Policy fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style Complete fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style Enforce fill:#ef4444,fill-opacity:0.15,stroke:#ef4444,stroke-width:2px,color:inherit
```

> [!NOTE] Regulatory Compliance
> Glacier Vault Lock enforces SEC Rule 17a-4(f), FINRA, and HIPAA data retention compliance where audit records must never be modified or erased before a retention term expires.
