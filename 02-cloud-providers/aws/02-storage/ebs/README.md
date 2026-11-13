# Amazon EBS (Elastic Block Store)

> **Focus**: Persistent block storage volumes, performance tiering, and snapshot mechanisms.

---

## 1. EBS Volume Attachment & Snapshots

EBS volumes are bound to a specific Availability Zone and attach to EC2 instances via high-speed NVMe protocols.

```mermaid
flowchart LR
    subgraph AZ["Availability Zone us-east-1a"]
        EC2["EC2 Instance"]
        EBS_VOL["EBS Volume<br/>(gp3 / io2)"]
        EC2 <== NVMe Bus ==> EBS_VOL
    end

    subgraph S3_REGION["Amazon S3 (Regional Fabric)"]
        SNAP1["Base Snapshot 1 (Full)"]
        SNAP2["Snapshot 2 (Incremental)"]
        SNAP1 -.-> SNAP2
    end

    EBS_VOL -->|CreateSnapshot API| SNAP1
    EBS_VOL -->|Delta Blocks Only| SNAP2

    style AZ fill:#0284c7,fill-opacity:0.08,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style EC2 fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:1.5px,color:inherit
    style EBS_VOL fill:#3b82f6,fill-opacity:0.15,stroke:#3b82f6,stroke-width:1.5px,color:inherit
    style S3_REGION fill:#f59e0b,fill-opacity:0.08,stroke:#f59e0b,stroke-width:1.5px,color:inherit
```

---

## 2. EBS Volume Types Breakdown

```mermaid
flowchart TD
    V_TYPE{"Volume Selection"}
    
    V_TYPE -->|SSD: General workloads, boots, databases| GP3["<b>gp3</b> (Recommended)<br/>Baseline 3,000 IOPS / 125 MB/s, independent scaling"]
    V_TYPE -->|SSD: Mission-critical, extreme IOPS| IO2["<b>io2 Block Express</b><br/>Sub-millisecond latency, up to 256,000 IOPS"]
    V_TYPE -->|HDD: High-throughput big data, log processing| ST1["<b>st1</b> (Throughput Optimized)<br/>Sequential I/O, cost-effective streaming"]
    V_TYPE -->|HDD: Cold data, lowest cost block storage| SC1["<b>sc1</b> (Cold HDD)<br/>Infrequently accessed massive datasets"]

    style V_TYPE fill:#64748b,fill-opacity:0.15,stroke:#64748b,stroke-width:1.5px,color:inherit
    style GP3 fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:1.5px,color:inherit
    style IO2 fill:#ef4444,fill-opacity:0.15,stroke:#ef4444,stroke-width:1.5px,color:inherit
    style ST1 fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style SC1 fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
```

> [!TIP] gp2 vs gp3
> Migrate legacy `gp2` to `gp3` to save 20% on storage costs while decoupling IOPS and throughput provisioning from disk capacity.
