# Amazon RDS (Relational Database Service)

> **Focus**: Managed relational engines, synchronous Multi-AZ high availability, and asynchronous Read Replicas.

---

## 1. Multi-AZ Standby vs. Read Replicas

```mermaid
flowchart TD
    APP["Application Compute Tier"]
    
    subgraph MULTI_AZ["Multi-AZ Synchronous HA (High Availability)"]
        direction LR
        PRI["Primary DB (AZ-a)<br/>(Active Read/Write)"]
        SEC["Standby DB (AZ-b)<br/>(Passive Sync Replica)"]
        PRI <== Synchronous Block Replication ==> SEC
    end

    subgraph READ_SCALE["Read Scaling Tier (Asynchronous)"]
        RR1["Read Replica 1 (AZ-a)"]
        RR2["Read Replica 2 (AZ-c)"]
    end

    APP -->|"Write Operations (DNS Endpoint)"| PRI
    APP -->|"Read Traffic (Reader Endpoint)"| RR1
    APP -->|"Read Traffic (Reader Endpoint)"| RR2
    PRI -.->|Async Replication| RR1
    PRI -.->|Async Replication| RR2

    style MULTI_AZ fill:#10b981,fill-opacity:0.08,stroke:#10b981,stroke-width:2px,color:inherit
    style READ_SCALE fill:#0284c7,fill-opacity:0.08,stroke:#0284c7,stroke-width:2px,color:inherit
```

---

## 2. Multi-AZ vs. Read Replica Comparison

| Dimension | Multi-AZ Deployment | Read Replicas |
| :--- | :--- | :--- |
| **Primary Goal** | High Availability & Disaster Recovery | Read Throughput Scaling |
| **Replication Type** | Synchronous (Zero data loss) | Asynchronous (Eventual consistency) |
| **Active DB Connection**| Primary only (Standby is inactive) | All Replicas accept direct read queries |
| **Failover Action** | Automatic DNS flip to standby (60-120s)| Manual or automated promotion to primary |
| **Cross-Region** | No (Single Region across AZs) | Yes (Cross-Region replicas supported) |
