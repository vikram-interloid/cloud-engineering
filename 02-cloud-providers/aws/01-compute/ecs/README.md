# Amazon ECS (Elastic Container Service) for Web Development

> **Focus**: Containerized web applications on AWS Fargate, ALB target groups, Blue/Green CodeDeploy traffic shifting, and internal microservice discovery.

---

## 1. Production Fargate Web Architecture

AWS Fargate executes Docker containers without provisioning or managing virtual machines, perfectly isolating web frontends, backend APIs, and background queue processors.

```mermaid
flowchart TD
    INTERNET["Web Users / Mobile Clients"] --> ALB["<b>Application Load Balancer</b><br/>(SSL Termination + Path Routing)"]

    subgraph ECS_CLUSTER["Amazon ECS Cluster (AWS Fargate)"]
        subgraph WEB_SVC["ECS Service: Web API (Desired: 3)"]
            T1["Task 1 (Container :8080)"]
            T2["Task 2 (Container :8080)"]
            T3["Task 3 (Container :8080)"]
        end

        subgraph WORKER_SVC["ECS Service: Background Workers"]
            W1["Worker Task (Consumes SQS)"]
            W2["Worker Task (Consumes SQS)"]
        end
    end

    ALB -->|Target Group :8080| WEB_SVC
    WEB_SVC -->|Push Slow Jobs| SQS["Amazon SQS Queue"]
    SQS --> WORKER_SVC
    WEB_SVC --> DB[("Amazon Aurora")]

    style INTERNET fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style ALB fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style ECS_CLUSTER fill:#10b981,fill-opacity:0.05,stroke:#10b981,stroke-width:2px,color:inherit
    style WEB_SVC fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:1.5px,color:inherit
    style WORKER_SVC fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
```

---

## 2. Blue/Green Canary Deployments with AWS CodeDeploy

Zero-downtime updates are achieved by spinning up replacement containers on a green target group and gradually shifting ALB listener traffic.

```mermaid
sequenceDiagram
    autonumber
    participant CD as AWS CodeDeploy
    participant ALB as Application Load Balancer
    participant Blue as Blue Target Group (v1 - Current)
    participant Green as Green Target Group (v2 - Replacement)

    CD->>Green: Launch v2 Tasks in Fargate
    Green->>ALB: Run Health Checks
    ALB-->>CD: Health Checks PASS
    
    rect rgb(245, 158, 11, 0.1)
    Note over ALB: CANARY TRAFFIC SHIFT (e.g. CodeDeployLinear10PercentEvery1Minute)
    CD->>ALB: Route 10% traffic to Green, 90% to Blue
    CD->>CD: Monitor CloudWatch Alarms (5XX Error Rate & Latency)
    CD->>ALB: Route 100% traffic to Green
    end

    Note over ALB,Blue: Blue Target Group Drained & v1 Tasks Terminated!
```

---

## 3. ECS Service Connect: Microservice-to-Microservice Communication

Internal web services (e.g., `checkout-service` calling `payment-service`) resolve each other using human-readable names without public internet exposure or custom DNS.

```mermaid
flowchart LR
    subgraph CLIENT_SVC["Web Frontend Service"]
        APP_C["Node.js Web App"] --> PROXY_C["Envoy Sidecar"]
    end

    subgraph BACKEND_SVC["Payment Service"]
        PROXY_B["Envoy Sidecar"] --> APP_B["Payment Microservice"]
    end

    APP_C -->|HTTP http://payment:8080| PROXY_C
    PROXY_C <== Encrypted mTLS Traffic ==> PROXY_B

    style CLIENT_SVC fill:#0284c7,fill-opacity:0.1,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style BACKEND_SVC fill:#10b981,fill-opacity:0.1,stroke:#10b981,stroke-width:1.5px,color:inherit
```

---

## 4. Production Task Definition JSON Structure

```json
{
  "family": "web-api-task",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::123456789012:role/webApiTaskRole",
  "containerDefinitions": [
    {
      "name": "web-container",
      "image": "123456789012.dkr.ecr.us-east-1.amazonaws.com/web-app:v2.1",
      "portMappings": [{ "containerPort": 8080, "protocol": "tcp" }],
      "secrets": [
        { "name": "DB_PASSWORD", "valueFrom": "arn:aws:secretsmanager:...:db_pass" }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/web-api",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```
