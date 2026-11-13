# Serverless Web Application Pattern

> **Focus**: Zero-server operations, automatic scale-to-zero, and utility pricing.

---

## 1. Full-Stack Serverless Architecture

```mermaid
flowchart TD
    USER["Client Browser / Mobile App"]

    subgraph FRONTEND["Frontend Distribution Tier"]
        R53["Route 53 DNS"]
        CF["CloudFront CDN"]
        S3["Amazon S3 (React / Vue SPA)"]
        USER --> R53 --> CF --> S3
    end

    subgraph AUTH["Identity Tier"]
        COG["Amazon Cognito User Pools"]
        USER -. 1. Authenticate (JWT) .-> COG
    end

    subgraph API_TIER["Backend API Tier"]
        APIGW["Amazon API Gateway (HTTP API)"]
        AUTH_VAL["Cognito JWT Authorizer"]
        USER -->|2. API Calls + Bearer Token| APIGW
        APIGW --- AUTH_VAL
    end

    subgraph COMPUTE["Compute & Persistence Tier"]
        LAMBDA["AWS Lambda (Node.js / Python MicroVMs)"]
        DDB[("Amazon DynamoDB (On-Demand)")]
        S3_UPLOADS[("S3 User Uploads Bucket")]

        APIGW --> LAMBDA
        LAMBDA --> DDB
        LAMBDA --> S3_UPLOADS
    end

    style FRONTEND fill:#0284c7,fill-opacity:0.08,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style AUTH fill:#8b5cf6,fill-opacity:0.1,stroke:#8b5cf6,stroke-width:1.5px,color:inherit
    style API_TIER fill:#10b981,fill-opacity:0.08,stroke:#10b981,stroke-width:1.5px,color:inherit
    style COMPUTE fill:#f59e0b,fill-opacity:0.08,stroke:#f59e0b,stroke-width:1.5px,color:inherit
```

---

## 2. Serverless Architectural Trade-Offs

- **Strengths**: Zero server management, built-in multi-AZ redundancy, scales from zero to 100k requests instantaneously.
- **Watch-Outs**: Lambda cold starts, database connection exhaustion (mitigate with RDS Proxy), max 15-minute execution limit.
