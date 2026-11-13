# Amazon Aurora for Web Development

> **Focus**: High-throughput relational web backends, Read Replicas for read-heavy web apps, and Aurora Serverless v2.

---

## 1. Web Application Read-Heavy Architecture (Writer vs. Reader Endpoints)

Web applications typically perform **80-90% read queries** (browsing catalogues, viewing profiles) and **10-20% write queries** (placing orders, editing settings). Aurora splits this traffic across dedicated endpoints.

```mermaid
flowchart TD
    subgraph WEB_APP["Web Compute Fleet (ECS / EKS / Lambda)"]
        direction LR
        POST_REQ["Write Requests (POST / PUT / DELETE)"]
        GET_REQ["Read Requests (GET / Search)"]
    end

    subgraph AURORA_ENDPOINTS["Aurora Cluster Endpoints"]
        CLUSTER_EP["<b>Cluster (Writer) Endpoint</b><br/>my-cluster.cluster-xyz.us-east-1.rds.amazonaws.com"]
        READER_EP["<b>Reader (Load-Balanced) Endpoint</b><br/>my-cluster.cluster-ro-xyz.us-east-1.rds.amazonaws.com"]
    end

    subgraph AURORA_INSTANCES["Aurora Database Instances"]
        WRITER["Primary DB (Writer Node)"]
        R1["Aurora Read Replica 1"]
        R2["Aurora Read Replica 2"]
        R3["Aurora Read Replica 3"]
    end

    POST_REQ --> CLUSTER_EP --> WRITER
    GET_REQ --> READER_EP
    READER_EP -. Round Robin .-> R1
    READER_EP -. Round Robin .-> R2
    READER_EP -. Round Robin .-> R3

    style WEB_APP fill:#0284c7,fill-opacity:0.08,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style AURORA_ENDPOINTS fill:#f59e0b,fill-opacity:0.1,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style WRITER fill:#10b981,fill-opacity:0.2,stroke:#10b981,stroke-width:2px,color:inherit
    style R1 fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style R2 fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style R3 fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
```

---

## 2. Aurora Serverless v2 for Spikey Web Traffic

Aurora Serverless v2 scales compute capacity instantly (in fine-grained ACUs) within milliseconds without dropping active TCP connections.

```mermaid
flowchart LR
    SPIKE["Sudden Flash Sale / HackerNews Frontpage"] --> ACU["Aurora ACU Scale-Up<br/>(e.g., 2 ACUs -> 32 ACUs in milliseconds)"]
    ACU --> STABLE["Zero Latency Spikes, Zero Connection Drops"]
    STABLE --> SCALE_DOWN["Traffic Drops -> Auto Scale-Down (Saves Budget)"]

    style SPIKE fill:#ef4444,fill-opacity:0.15,stroke:#ef4444,stroke-width:2px,color:inherit
    style ACU fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style SCALE_DOWN fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
```
