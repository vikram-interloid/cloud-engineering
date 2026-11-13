# Amazon EC2 for Web Development

> **Focus**: Production web server setup, Nginx reverse proxy, systemd process management, zero-downtime rolling updates, and stateless session scaling.

---

## 1. Production Host Architecture (Nginx + Node/Python Backend)

A production EC2 web instance runs **Nginx** as a high-performance reverse proxy for TLS termination, gzip compression, and static asset caching, forwarding dynamic requests to internal web application processes via Unix sockets or localhost ports.

```mermaid
flowchart TD
    ALB["Application Load Balancer<br/>(Health Checks on :80)"] -->|HTTP Forward| NGINX["<b>Nginx Reverse Proxy</b> (:80 / :443)"]

    subgraph EC2_INSTANCE["Amazon EC2 Web Host"]
        direction TB
        NGINX -->|Proxy Pass /api/| SOCK["Unix Domain Socket / Localhost :3000"]
        NGINX -->|Static Files /public/| DISK["Local NVMe EBS Cache"]
        
        subgraph PROCESS_MGR["systemd Process Supervision"]
            SOCK --> APP1["Node.js / Python Gunicorn Worker 1"]
            SOCK --> APP2["Node.js / Python Gunicorn Worker 2"]
            SOCK --> APP3["Node.js / Python Gunicorn Worker 3"]
        end
        
        APP1 -.-> IAM_CREDS["IAM Instance Profile (Auto-rotated STS tokens)"]
    end

    APP1 -->|Shared State| REDIS[("Amazon ElastiCache Redis<br/>(User Sessions & Rate Limits)")]
    APP1 -->|Persistence| RDS[("Amazon Aurora PostgreSQL")]

    style ALB fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style EC2_INSTANCE fill:#10b981,fill-opacity:0.06,stroke:#10b981,stroke-width:2px,color:inherit
    style NGINX fill:#10b981,fill-opacity:0.2,stroke:#10b981,stroke-width:1.5px,color:inherit
    style PROCESS_MGR fill:#f59e0b,fill-opacity:0.08,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style REDIS fill:#ef4444,fill-opacity:0.15,stroke:#ef4444,stroke-width:1.5px,color:inherit
    style RDS fill:#3b82f6,fill-opacity:0.15,stroke:#3b82f6,stroke-width:1.5px,color:inherit
```

---

## 2. Zero-Downtime Rolling Deployment with ALB

When deploying new code across an Auto Scaling Group (ASG), the Application Load Balancer coordinates with **Connection Draining (Deregistration Delay)** so active HTTP requests finish before instances terminate.

```mermaid
sequenceDiagram
    autonumber
    participant ASG as Auto Scaling Group
    participant ALB as Application Load Balancer
    participant OldEC2 as v1 Instance (Old)
    participant NewEC2 as v2 Instance (New)

    Note over ASG: Deployment Triggered (New Launch Template)
    ASG->>NewEC2: Launch v2 Instance & Run UserData Boot
    NewEC2->>ALB: Register with Target Group
    loop ALB Health Checks
        ALB->>NewEC2: GET /health (Port 80)
        NewEC2-->>ALB: 200 OK (Healthy)
    end
    ALB->>NewEC2: Shift 50% Live Web Traffic to v2
    
    rect rgb(239, 68, 68, 0.1)
    Note over ALB,OldEC2: CONNECTION DRAINING (e.g. 60 seconds)
    ALB->>OldEC2: Stop sending NEW requests
    OldEC2-->>ALB: Complete in-flight HTTP requests
    end
    
    ASG->>OldEC2: Terminate v1 Instance
    Note over ALB,NewEC2: 100% Traffic running on v2 with ZERO dropped packets!
```

---

## 3. Stateless Web Tier vs Sticky Sessions

```mermaid
flowchart LR
    subgraph BAD["Anti-Pattern: Sticky Local Sessions"]
        direction TB
        B_ALB["ALB (Sticky Cookie)"] --> B_EC1["EC2 A: Session Memory"]
        B_ALB --> B_EC2["EC2 B: Session Memory"]
        Note1["If EC2 A crashes, users are logged out!"]
    end

    subgraph GOOD["Production Pattern: Stateless with Redis"]
        direction TB
        G_ALB["ALB (Round Robin)"] --> G_EC1["EC2 A (Stateless)"]
        G_ALB --> G_EC2["EC2 B (Stateless)"]
        G_EC1 --> REDIS_HUB[("ElastiCache Redis Session Store")]
        G_EC2 --> REDIS_HUB
        Note2["Any instance can crash or scale with zero session loss!"]
    end

    style BAD fill:#ef4444,fill-opacity:0.08,stroke:#ef4444,stroke-width:1.5px,color:inherit
    style GOOD fill:#10b981,fill-opacity:0.08,stroke:#10b981,stroke-width:2px,color:inherit
```

---

## 4. Production Web Server User Data Script

```bash
#!/bin/bash
# Amazon Linux 2023 - Web Server Automated Bootstrap
dnf update -y
dnf install -y nginx git nodejs

# Configure Nginx Reverse Proxy
cat << 'CONF' > /etc/nginx/conf.d/webapp.conf
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
    }
}
CONF

systemctl restart nginx
systemctl enable nginx
```
