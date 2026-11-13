# 02 · AWS Storage Services

> **Module**: Storage  
> **Focus**: Data persistence models: Object, Block, File, and Long-Term Cold Archive.

---

## Storage Types Spectrum

```mermaid
flowchart TD
    DATA["Data Storage Requirements"]
    
    DATA -->|REST API Object Storage & Data Lake| S3["<b>Amazon S3</b><br/>Unstructured, unlimited scale, 11 9s durability"]
    DATA -->|Low-latency Virtual Hard Disks for EC2| EBS["<b>Amazon EBS</b><br/>Block storage, NVMe, snapshotable, single-AZ"]
    DATA -->|POSIX Shared File System for multiple EC2/Containers| EFS["<b>Amazon EFS</b><br/>Elastic NFSv4, multi-AZ, scales automatically"]
    DATA -->|Long-term compliance & cold data archiving| GLACIER["<b>Amazon S3 Glacier</b><br/>Ultra-low cost, flexible retrieval hours"]

    style DATA fill:#64748b,fill-opacity:0.15,stroke:#64748b,stroke-width:1.5px,color:inherit
    style S3 fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style EBS fill:#3b82f6,fill-opacity:0.15,stroke:#3b82f6,stroke-width:2px,color:inherit
    style EFS fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:2px,color:inherit
    style GLACIER fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
```

---

## Storage Comparison Matrix

| Service | Paradigm | Protocol | Concurrency | Durability |
| :--- | :--- | :--- | :--- | :--- |
| **Amazon S3** | Object (Key-Value) | HTTPS / REST | Millions concurrent | 99.999999999% (11 9s) |
| **Amazon EBS** | Block Storage | NVMe / FC | Single EC2 (or Multi-Attach) | 99.8% – 99.999% |
| **Amazon EFS** | Network File (POSIX) | NFSv4.1 | Thousands of instances | 99.999999999% (11 9s) |
| **S3 Glacier** | Archive Object | HTTPS / REST | Batch / Async job | 99.999999999% (11 9s) |

---

## Storage Directory Index

- **[[s3/README|S3]]** — Simple Storage Service, storage classes, lifecycle policies, and bucket security.
- **[[ebs/README|EBS]]** — Elastic Block Store, volume types (`gp3`, `io2`), and snapshot management.
- **[[efs/README|EFS]]** — Elastic File System, multi-AZ POSIX mounting, and throughput modes.
- **[[glacier/README|Glacier]]** — Deep archival, retrieval tiers, and compliance Vault Locks.
