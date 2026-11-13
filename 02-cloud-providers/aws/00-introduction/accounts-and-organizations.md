# AWS Accounts & Organizations Architecture

> **Focus**: Multi-account security boundaries, organizational units (OUs), and Service Control Policies (SCPs).

---

## 1. Enterprise Multi-Account Topology

A single AWS account is an administrative, blast-radius, and billing boundary. Enterprise environments organize accounts under **AWS Organizations**.

```mermaid
flowchart TD
    ROOT["<b>Management (Root) Account</b><br/>Consolidated Billing & Root SCPs"]

    ROOT --> OU_CORE["<b>Core / Infrastructure OU</b>"]
    ROOT --> OU_SEC["<b>Security OU</b>"]
    ROOT --> OU_WORKLOADS["<b>Workloads OU</b>"]
    ROOT --> OU_SANDBOX["<b>Sandbox OU</b>"]

    OU_CORE --> ACC_NET["Network Hub Account<br/>(Transit Gateway, Direct Connect)"]
    OU_CORE --> ACC_SHARED["Shared Services Account<br/>(CI/CD, Artifacts, Directory)"]

    OU_SEC --> ACC_LOG["Log Archive Account<br/>(Central S3, WORM Vault)"]
    OU_SEC --> ACC_AUDIT["Security Tooling Account<br/>(GuardDuty, SecurityHub, IAM Identity)"]

    OU_WORKLOADS --> OU_PROD["Prod OU"]
    OU_WORKLOADS --> OU_NONPROD["Non-Prod OU"]

    OU_PROD --> ACC_PROD["App Production Account"]
    OU_NONPROD --> ACC_DEV["App Dev / Staging Account"]

    style ROOT fill:#dc2626,fill-opacity:0.12,stroke:#dc2626,stroke-width:2px,color:inherit
    style OU_CORE fill:#0284c7,fill-opacity:0.1,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style OU_SEC fill:#ef4444,fill-opacity:0.1,stroke:#ef4444,stroke-width:1.5px,color:inherit
    style OU_WORKLOADS fill:#10b981,fill-opacity:0.1,stroke:#10b981,stroke-width:1.5px,color:inherit
    style OU_SANDBOX fill:#8b5cf6,fill-opacity:0.1,stroke:#8b5cf6,stroke-width:1.5px,color:inherit
```

---

## 2. Service Control Policy (SCP) Guardrails

SCPs define the maximum allowed permissions for member accounts. They act as **guardrails**, not grants.

```mermaid
flowchart LR
    subgraph SCP_GUARD["SCP Boundary (Deny S3 Delete / Restrict Regions)"]
        direction TB
        subgraph IAM_PERM["IAM Identity Permissions (Admin Access)"]
            Action["Effective Permitted Actions:<br/>(Intersection of SCP AND IAM)"]
        end
    end

    style SCP_GUARD fill:#ef4444,fill-opacity:0.08,stroke:#ef4444,stroke-width:2px,color:inherit
    style IAM_PERM fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:1.5px,color:inherit
```

> [!IMPORTANT] SCP Golden Rules
> 1. An explicit **Deny** in an SCP overrides any IAM policy in member accounts (even the Root user).
> 2. The Management Account is **not** restricted by SCPs.
> 3. Use SCPs to enforce compliance (e.g., restrict allowed AWS regions, prevent disabling CloudTrail).
