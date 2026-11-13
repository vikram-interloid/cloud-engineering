# Amazon API Gateway for Web Development

> **Focus**: REST & HTTP APIs for Single Page Applications, CORS preflight handling, Cognito/JWT authorizers, and WebSockets for real-time applications.

---

## 1. Web Application Authorization & Request Lifecycle

API Gateway validates user tokens at the edge **before** triggering any compute or database operations.

```mermaid
sequenceDiagram
    autonumber
    actor SPA as React / Vue Frontend
    participant Cognito as Amazon Cognito (User Pool)
    participant APIGW as Amazon API Gateway (HTTP API)
    participant Lambda as Backend Lambda Function
    participant DB as DynamoDB / RDS

    SPA->>Cognito: 1. Login with Username & Password
    Cognito-->>SPA: 2. Return JWT ID & Access Tokens
    
    rect rgb(2, 132, 199, 0.1)
    Note over SPA,APIGW: 3. Authenticated Web Request
    SPA->>APIGW: GET /api/orders (Authorization: Bearer <JWT>)
    APIGW->>APIGW: 4. Native JWT Authorizer verifies signature & expiry
    end

    alt Token Invalid / Expired
        APIGW-->>SPA: 401 Unauthorized (Zero Lambda execution cost!)
    else Token Valid
        APIGW->>Lambda: 5. Invoke with event.requestContext.authorizer.jwt.claims
        Lambda->>DB: 6. Fetch user orders
        DB-->>Lambda: Records
        Lambda-->>APIGW: 200 OK + JSON
        APIGW-->>SPA: 200 OK
    end
```

---

## 2. Cross-Origin Resource Sharing (CORS) Preflight Handling

When a frontend calls an API on a different domain, browsers automatically dispatch an `OPTIONS` preflight request.

```mermaid
sequenceDiagram
    autonumber
    actor Browser as Web Browser (Frontend)
    participant APIGW as API Gateway (CORS Engine)
    participant Lambda as Backend Lambda

    Browser->>APIGW: OPTIONS /api/checkout (Origin: https://app.example.com)
    Note over APIGW: API Gateway terminates preflight instantly!
    APIGW-->>Browser: 204 No Content<br/>Access-Control-Allow-Origin: https://app.example.com<br/>Access-Control-Allow-Methods: GET,POST,OPTIONS<br/>Access-Control-Allow-Headers: Authorization,Content-Type
    
    Browser->>APIGW: POST /api/checkout { cartId: "123" }
    APIGW->>Lambda: Forward Request
    Lambda-->>APIGW: 200 OK + Access-Control-Allow-Origin
    APIGW-->>Browser: 200 OK (Payment Processed)
```

---

## 3. Real-Time WebSockets for Live Web Chat & Notifications

Unlike standard stateless HTTP, API Gateway WebSocket APIs maintain persistent, bi-directional connections to thousands of browser clients.

```mermaid
flowchart TD
    CLIENT["Web Browser (new WebSocket('wss://api.example.com'))"]
    
    subgraph WSS_ENGINE["API Gateway WebSocket API"]
        ROUTE_CONN["$connect Route"]
        ROUTE_DISC["$disconnect Route"]
        ROUTE_MSG["Custom Routes (e.g. 'sendMessage')"]
    end

    subgraph PERSISTENCE["Connection State Store"]
        DDB[("Amazon DynamoDB<br/>(connectionId, userId)")]
    end

    subgraph HANDLERS["Lambda Handlers"]
        LAMBDA_CONN["Connect Lambda (Auth & Store ID)"]
        LAMBDA_MSG["Message Broadcast Lambda"]
    end

    CLIENT <== Persistent Full-Duplex Socket ==> WSS_ENGINE
    ROUTE_CONN --> LAMBDA_CONN
    LAMBDA_CONN --> DDB
    ROUTE_MSG --> LAMBDA_MSG
    LAMBDA_MSG -->|Scan Connections| DDB
    LAMBDA_MSG -.->|"PostToConnection API (Push back)"| WSS_ENGINE

    style CLIENT fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style WSS_ENGINE fill:#f59e0b,fill-opacity:0.1,stroke:#f59e0b,stroke-width:2px,color:inherit
    style PERSISTENCE fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style HANDLERS fill:#0284c7,fill-opacity:0.1,stroke:#0284c7,stroke-width:1.5px,color:inherit
```

---

## 4. HTTP API vs. REST API for Web Developers

| Feature | HTTP API (Recommended for Web) | REST API |
| :--- | :--- | :--- |
| **Cost** | **$1.00** per million requests | **$3.50** per million requests |
| **Latency** | Sub-10ms (Optimized engine) | ~30-50ms |
| **Native JWT / OIDC** | Built-in (Cognito, Auth0, Okta) | Requires custom Lambda Authorizer |
| **CORS Configuration**| 1-Click native configuration | Requires Mock Integration setup |
| **Advanced Features** | No WAF, no API keys, no cache | WAF, API Keys, Usage Plans, Request Validation |
