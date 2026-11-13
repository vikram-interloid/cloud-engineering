# Amazon S3 (Simple Storage Service) for Web Development

> **Focus**: Static web hosting, Single Page Application (SPA) routing, Origin Access Control (OAC), Presigned URLs, and event-driven media processing.

---

## 1. Single Page Application (SPA) Hosting Architecture

Hosting modern frontend applications (React, Vue, Angular, Next.js static export) on AWS pairs S3 for asset storage with CloudFront for global caching, SSL/TLS, and Origin Access Control (OAC).

```mermaid
flowchart TD
    CLIENT["Browser Client<br/>(https://app.example.com)"]
    
    subgraph EDGE["Global Edge (Amazon CloudFront)"]
        CF["CloudFront CDN Distribution"]
        OAC["Origin Access Control (OAC)<br/>Signs requests with SigV4"]
        ACM["AWS Certificate Manager (ACM)<br/>Custom SSL/TLS (*.example.com)"]
        CF --- OAC
        CF --- ACM
    end

    subgraph S3_BUCKET["Private Amazon S3 Bucket"]
        BPA["S3 Block Public Access (ENABLED)"]
        INDEX["index.html (SPA Entrypoint)"]
        STATIC["/assets/* (JS, CSS, SVGs, Fonts)"]
        POLICY["Bucket Policy:<br/>Allow ONLY CloudFront OAC Principal"]
        
        BPA --- POLICY
        POLICY --> INDEX
        POLICY --> STATIC
    end

    CLIENT -->|1. HTTPS Request| CF
    CF -->|2. Signed Fetch via OAC| S3_BUCKET
    CLIENT -.->|"Direct S3 URL Blocked (403)"| S3_BUCKET

    style CLIENT fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style EDGE fill:#0284c7,fill-opacity:0.08,stroke:#0284c7,stroke-width:2px,color:inherit
    style S3_BUCKET fill:#10b981,fill-opacity:0.08,stroke:#10b981,stroke-width:2px,color:inherit
    style OAC fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
```

---

## 2. SPA Client-Side Routing & 404 Fallback

Client-side routers (React Router, Vue Router) use HTML5 `history.pushState()`. When a user refreshes `/dashboard/settings`, S3 does not have a physical key named `dashboard/settings`. CloudFront custom error responses rewrite `403/404` to `index.html` with a `200 OK`.

```mermaid
sequenceDiagram
    autonumber
    actor User as Web Browser
    participant CF as CloudFront Distribution
    participant S3 as Amazon S3 (Private Bucket)

    User->>CF: GET /dashboard/settings
    CF->>S3: GET /dashboard/settings
    S3-->>CF: 403 / 404 Not Found (Object does not exist)
    
    rect rgb(16, 185, 129, 0.1)
    Note over CF: CloudFront Custom Error Response:<br/>Map 403/404 -> /index.html with HTTP 200 OK
    end

    CF->>S3: GET /index.html
    S3-->>CF: Return index.html (200 OK)
    CF-->>User: 200 OK + index.html
    Note over User: React / Vue Router boots and mounts<br/>the Settings view in the browser!
```

---

## 3. Presigned URLs: Direct Browser-to-S3 Uploads

Traditional web architectures funnel user file uploads through web backend servers, saturating CPU, memory, and bandwidth. With **Presigned URLs**, the web server simply generates a temporary signed cryptographic URL; the browser uploads directly to S3.

```mermaid
sequenceDiagram
    autonumber
    actor Browser as Web Browser (Frontend)
    participant API as Web API Server (Node / Python / Go)
    participant S3 as Amazon S3 Bucket

    Browser->>API: 1. POST /api/upload-request { filename: "avatar.png", mime: "image/png" }
    Note over API: Authenticate user & validate permissions
    API->>API: 2. Generate S3 Presigned URL (PutObject, TTL = 5 mins)
    API-->>Browser: 3. Return { uploadUrl: "https://my-bucket.s3.amazonaws.com/avatar.png?AWSAccessKeyId=..." }
    
    rect rgb(16, 185, 129, 0.1)
    Browser->>S3: 4. PUT /avatar.png (Binary payload streamed directly)
    S3-->>Browser: 5. 200 OK (Uploaded without server bottleneck!)
    end
    
    Browser->>API: 6. POST /api/confirm-upload { filename: "avatar.png" }
    API->>API: 7. Save metadata in Database
```

---

## 4. Asynchronous Event-Driven Media Processing

Once an object lands in S3, **S3 Event Notifications** trigger background serverless workers without holding up web users.

```mermaid
flowchart LR
    BROWSER["Browser Upload"] -->|Direct PUT| S3_RAW["S3 Bucket: raw-uploads/"]
    
    S3_RAW -->|s3:ObjectCreated:* Event| LAMBDA["AWS Lambda<br/>(Sharp / Pillow Image Resizer)"]
    
    LAMBDA -->|Save Optimized Thumbnails| S3_OPT["S3 Bucket: optimized-images/"]
    LAMBDA -->|Store Dimensions & CDN URLs| DDB[("Amazon DynamoDB")]
    LAMBDA -->|Push Notification / WebSocket| BROWSER

    style BROWSER fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style S3_RAW fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style LAMBDA fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style S3_OPT fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style DDB fill:#8b5cf6,fill-opacity:0.15,stroke:#8b5cf6,stroke-width:1.5px,color:inherit
```

---

## 5. Web Developer Production Checklist

| Feature | Production Setting | Why it Matters |
| :--- | :--- | :--- |
| **Origin Access Control (OAC)** | Enabled on S3 Bucket Policy | Prevents bypassing CloudFront, forces WAF & CDN inspection |
| **Block Public Access** | All 4 checkboxes **ON** | Prevents data leaks; S3 is only readable via CloudFront |
| **CORS Configuration** | Allow origin `https://app.example.com`, Methods: `GET, PUT, HEAD` | Allows SPA to download fonts & make direct PUT uploads |
| **Cache-Control Headers** | `max-age=31536000, immutable` for hashed assets (`/assets/*.js`) | Browsers & CDN cache hashed assets forever |
| **index.html Caching** | `max-age=0, must-revalidate` for `index.html` | Ensures users immediately get new app deployments |
