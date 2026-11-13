# Amazon Route 53 for Web Development

> **Focus**: Production domain routing, zone apex Alias records to CloudFront/ALB, ACM SSL validation, and multi-region failover.

---

## 1. Web Domain Resolution & Alias Routing Architecture

Standard DNS protocols forbid `CNAME` records at the root apex domain (`example.com`). Route 53 provides **Alias records**—an AWS-specific virtual DNS record that maps apex domains directly to CloudFront or ALB with zero performance penalty and free queries.

```mermaid
flowchart TD
    CLIENT["Web Browser Client"] --> R53["<b>Amazon Route 53 Hosted Zone</b><br/>(example.com)"]

    subgraph ROUTING["Route 53 DNS Records"]
        direction TB
        APEX["<b>Alias 'A' Record: example.com</b><br/>Target: d111111abcdef8.cloudfront.net"]
        SUB_WWW["<b>CNAME Record: www.example.com</b><br/>Target: example.com (Redirect / Alias)"]
        SUB_API["<b>Alias 'A' Record: api.example.com</b><br/>Target: dualstack.my-alb-123.us-east-1.elb.amazonaws.com"]
        ACM_REC["<b>CNAME Record: _a7b8c9.example.com</b><br/>Target: _x1y2z3.acm-validations.aws (SSL Auto-Renewal)"]
    end

    R53 --> APEX
    R53 --> SUB_WWW
    R53 --> SUB_API
    R53 --> ACM_REC

    APEX --> CF["Amazon CloudFront (Frontend React SPA)"]
    SUB_API --> ALB["Application Load Balancer (Backend API)"]
    ACM_REC --> ACM["AWS Certificate Manager (Free Auto-Renewing SSL)"]

    style CLIENT fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style R53 fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style ROUTING fill:#f59e0b,fill-opacity:0.06,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style CF fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style ALB fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
```

---

## 2. Automated SSL/TLS Validation with ACM & Route 53

```mermaid
sequenceDiagram
    autonumber
    actor Dev as Web Developer / CI
    participant ACM as AWS Certificate Manager
    participant R53 as Route 53 Hosted Zone
    participant CA as AWS Certificate Authority

    Dev->>ACM: Request Public Certificate (*.example.com)
    ACM-->>Dev: Validation CNAME Record Provided
    Dev->>R53: Create CNAME: _acm_challenge.example.com -> _validation.aws
    loop Automated DNS Validation
        CA->>R53: Query _acm_challenge.example.com
        R53-->>CA: Return validation secret
    end
    CA-->>ACM: Domain Ownership Confirmed!
    ACM->>ACM: Issue Certificate & Auto-Renew Yearly Forever
```

---

## 3. Active-Passive Web Disaster Recovery (DNS Failover)

```mermaid
flowchart TD
    USERS["Global Web Users"] --> R53["Route 53 Failover Routing Policy"]

    subgraph PRIMARY["Primary Production Region (us-east-1)"]
        ALB_PRI["Primary ALB + Web Cluster"]
        HC["Route 53 Health Check<br/>(Polls https://api.example.com/health every 10s)"]
        HC -. Monitors .-> ALB_PRI
    end

    subgraph DR_REGION["Disaster Recovery Region (us-west-2)"]
        S3_MAINT["S3 Static Maintenance Web Page<br/>('System undergoing emergency maintenance')"]
    end

    R53 -->|1. Healthy: 100% Traffic| ALB_PRI
    R53 -.->|2. Unhealthy: Auto-Reroute to Maintenance Page| S3_MAINT

    style USERS fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style R53 fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style PRIMARY fill:#10b981,fill-opacity:0.08,stroke:#10b981,stroke-width:1.5px,color:inherit
    style DR_REGION fill:#ef4444,fill-opacity:0.08,stroke:#ef4444,stroke-width:1.5px,color:inherit
```

---

## 4. Web Developer DNS Cheat Sheet

| Domain Query | Record Type | Value / Routing Policy | Notes |
| :--- | :--- | :--- | :--- |
| `example.com` | **A (Alias)** | `d123.cloudfront.net` (Evaluate Target Health: Yes) | Free apex resolution |
| `www.example.com` | **CNAME** or **A (Alias)** | `example.com` | Canonical web URL |
| `api.example.com` | **A (Alias)** | ALB DNS name | Direct to API load balancer |
| `staging.example.com` | **A (Alias)** | Staging ALB / CloudFront | Test environment |
