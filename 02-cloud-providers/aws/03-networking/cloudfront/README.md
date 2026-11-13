# Amazon CloudFront for Web Development

> **Focus**: Global edge acceleration, single-domain frontend + backend routing, security headers with edge compute, and cache invalidation.

---

## 1. Unified Multi-Origin Web Application Architecture

Modern web applications avoid Cross-Origin Resource Sharing (CORS) complexity by using CloudFront to serve both the frontend SPA and the dynamic backend APIs under a **single domain** (`https://app.example.com`).

```mermaid
flowchart TD
    CLIENT["Web / Mobile Browser<br/>(https://app.example.com)"] --> CF["<b>Amazon CloudFront Distribution</b><br/>(Custom Domain + ACM SSL)"]

    subgraph BEHAVIORS["Path-Based Cache Behaviors"]
        direction TB
        B_API["<b>Path: /api/*</b><br/>Cache Policy: Disabled (TTL=0)<br/>Forward All Headers, Cookies, Query Params"]
        B_STATIC["<b>Path: /assets/*</b><br/>Cache Policy: Optimized (TTL=1 Year)<br/>Compression: Gzip + Brotli"]
        B_DEFAULT["<b>Path: Default (*)</b><br/>Cache Policy: CachingOptimized (TTL=5m)<br/>Error Rewrite: 404 -> /index.html"]
    end

    subgraph ORIGINS["Backend Origins"]
        ALB["<b>Origin 1: ALB / API Gateway</b><br/>Node.js / Python / Go APIs"]
        S3["<b>Origin 2: S3 Private Bucket</b><br/>SPA React/Vue Bundles via OAC"]
    end

    CF --> BEHAVIORS
    B_API -->|Forward Live Request| ALB
    B_STATIC -->|Cache & Serve| S3
    B_DEFAULT -->|Serve index.html| S3

    style CLIENT fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style CF fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style BEHAVIORS fill:#f59e0b,fill-opacity:0.08,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style ALB fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style S3 fill:#3b82f6,fill-opacity:0.15,stroke:#3b82f6,stroke-width:2px,color:inherit
```

---

## 2. Edge Security & Header Manipulation with CloudFront Functions

CloudFront Functions run lightweight JavaScript at **218+ edge PoPs** in `< 1 millisecond`. In web development, they inject critical security headers (HSTS, CSP, X-Frame-Options) and normalize URLs.

```mermaid
sequenceDiagram
    autonumber
    actor Client as Browser
    participant Edge as CloudFront PoP (Edge)
    participant CFF as CloudFront Function (Viewer Response)
    participant Origin as S3 / ALB Origin

    Client->>Edge: GET /profile
    Edge->>Origin: Fetch Response
    Origin-->>Edge: 200 OK (Raw HTML)
    
    rect rgb(16, 185, 129, 0.1)
    Edge->>CFF: viewer-response hook
    Note over CFF: Inject Security Headers:<br/>Strict-Transport-Security: max-age=63072000<br/>X-Content-Type-Options: nosniff<br/>X-Frame-Options: DENY<br/>Content-Security-Policy: default-src 'self'
    CFF-->>Edge: Modified Response Headers
    end

    Edge-->>Client: 200 OK + Hardened Security Headers
```

---

## 3. Cache Invalidation & Zero-Downtime Deployment Lifecycle

When deploying a new frontend build, never delete the bucket. Use content hashing and an automated cache invalidation step in your CI/CD pipeline.

```mermaid
flowchart LR
    BUILD["1. CI/CD Build<br/>(npm run build)"] --> UPLOAD["2. Upload to S3<br/>/assets/bundle.a1b2c3.js<br/>/index.html"]
    UPLOAD --> INVALIDATE["3. Invalidate CloudFront Cache<br/>aws cloudfront create-invalidation<br/>--paths '/index.html'"]
    INVALIDATE --> CLIENT["4. Browser Fetches Fresh index.html<br/>which references new hashed bundles!"]

    style BUILD fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style UPLOAD fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style INVALIDATE fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:2px,color:inherit
    style CLIENT fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:1.5px,color:inherit
```

> [!TIP] FinOps & Cache Invalidation Best Practice
> Only invalidate `"/index.html"` during deployments. Because JavaScript and CSS bundles contain unique content hashes (`main.[hash].js`), they never need invalidation and can be cached by browsers indefinitely.
