# Amazon SQS for Web Development

> **Focus**: Decoupling slow web operations, asynchronous background workers, consumer autoscaling by backlog, and Dead Letter Queues (DLQ).

---

## 1. Web Request Decoupling: Synchronous API vs. SQS Worker

Web servers must reply to users within `< 200 milliseconds`. Heavy operations (sending verification emails, generating PDFs, processing billing) should be offloaded to an asynchronous SQS queue.

```mermaid
flowchart TD
    subgraph SYNC["Synchronous Anti-Pattern (Slow & Fragile)"]
        direction LR
        S_USER["User"] -->|POST /order| S_API["Web Server"]
        S_API --> S_EMAIL["Send Email (3s)"]
        S_EMAIL --> S_PDF["Generate Invoice (5s)"]
        S_PDF --> S_PAY["Process Payment (2s)"]
        S_PAY -->>|Total: 10s Latency!| S_USER
    end

    subgraph ASYNC["Decoupled Asynchronous Pattern (Fast & Resilient)"]
        direction LR
        A_USER["User"] -->|POST /order| A_API["Web Server"]
        A_API -->|1. Enqueue Message| QUEUE[("<b>Amazon SQS</b>")]
        A_API -->>|"2. Immediate 202 Accepted (50ms)"| A_USER
        QUEUE --> WORKER["Background Worker Pool<br/>(Processes Email, PDF, Payment in Background)"]
    end

    style SYNC fill:#ef4444,fill-opacity:0.08,stroke:#ef4444,stroke-width:1.5px,color:inherit
    style ASYNC fill:#10b981,fill-opacity:0.08,stroke:#10b981,stroke-width:2px,color:inherit
```

---

## 2. Queue Backlog Autoscaling Loop (Target Tracking)

Instead of scaling workers on CPU, web worker fleets scale based on the queue depth metric: **`BacklogPerWorker = ApproximateNumberOfMessagesVisible / NumberOfWorkers`**.

```mermaid
flowchart LR
    QUEUE[("Amazon SQS Queue")] -->|Reports Metrics| CW["CloudWatch Metric:<br/>ApproximateNumberOfMessagesVisible"]
    
    CW --> SCALE_POLICY{"BacklogPerWorker &gt; 100?"}
    
    SCALE_POLICY -->|Yes| ASG["Scale Out Worker Fleet<br/>(ECS Fargate / EC2 ASG)"]
    SCALE_POLICY -->|"No (Queue empty)"| IDLE["Scale Down to Minimum Tasks<br/>(Save Compute Costs)"]

    style QUEUE fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:2px,color:inherit
    style CW fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style ASG fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
```

---

## 3. Dead Letter Queue (DLQ) & Automated Redrive

When bad payloads or downstream API outages cause processing failures, messages move to a DLQ without blocking other valid web orders.

```mermaid
sequenceDiagram
    autonumber
    participant Queue as SQS Main Queue
    participant Worker as Worker Process
    participant DLQ as SQS Dead Letter Queue (DLQ)
    participant Dev as Engineering Team

    Queue->>Worker: ReceiveMessage (Attempt 1)
    Worker->>Worker: Fails (Third-party API 500 error)
    Queue->>Worker: ReceiveMessage (Attempt 2)
    Worker->>Worker: Fails
    Queue->>Worker: ReceiveMessage (Attempt 3: maxReceiveCount reached)
    Worker->>Worker: Fails
    
    rect rgb(239, 68, 68, 0.1)
    Queue->>DLQ: Automatically route message to DLQ
    DLQ->>Dev: CloudWatch Alarm fires!
    end

    Note over Dev: Bug fixed / Downstream service restored
    Dev->>DLQ: Trigger SQS StartMessageMoveTask (Redrive back to Main Queue)
    DLQ->>Queue: Messages safely reprocessed!
```
