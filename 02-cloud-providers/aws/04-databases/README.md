# 04 · AWS Database Services

> **Module**: Databases  
> **Focus**: Relational, Cloud-Native, NoSQL Key-Value, In-Memory Caching, and Columnar Data Warehouses.

---

## AWS Purpose-Built Database Matrix

```mermaid
flowchart TD
    REQ{"Workload & Data Model Requirements"}
    
    REQ -->|Structured ACID, standard SQL, enterprise engines| RDS["<b>Amazon RDS</b><br/>PostgreSQL, MySQL, MariaDB, Oracle, SQL Server"]
    REQ -->|High-throughput cloud-native relational, 5x MySQL perf| AURORA["<b>Amazon Aurora</b><br/>Serverless v2, 6-way storage replication, global DB"]
    REQ -->|Single-digit ms latency at any scale, Key-Value / Document| DDB["<b>Amazon DynamoDB</b><br/>Fully managed NoSQL, Global Tables, infinite scale"]
    REQ -->|Sub-millisecond microsecond in-memory caching| CACHE["<b>Amazon ElastiCache</b><br/>Redis & Memcached, session store, real-time leaderboard"]
    REQ -->|Complex SQL OLAP analytics, petabyte-scale warehousing| RS["<b>Amazon Redshift</b><br/>Columnar storage, AQUA acceleration, Serverless"]

    style REQ fill:#64748b,fill-opacity:0.15,stroke:#64748b,stroke-width:1.5px,color:inherit
    style RDS fill:#3b82f6,fill-opacity:0.15,stroke:#3b82f6,stroke-width:1.5px,color:inherit
    style AURORA fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style DDB fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style CACHE fill:#ef4444,fill-opacity:0.15,stroke:#ef4444,stroke-width:1.5px,color:inherit
    style RS fill:#8b5cf6,fill-opacity:0.15,stroke:#8b5cf6,stroke-width:1.5px,color:inherit
```

---

## Database Directory Index

- **[[rds/README|RDS]]** — Relational Database Service, Multi-AZ automated failover, and Read Replicas.
- **[[aurora/README|Aurora]]** — Cloud-native distributed storage engine, Aurora Serverless v2, and Global Database.
- **[[dynamodb/README|DynamoDB]]** — Serverless NoSQL, Partition Keys, Global Secondary Indexes, and Streams.
- **[[elasticache/README|ElastiCache]]** — In-memory Redis/Memcached acceleration and cache-aside patterns.
- **[[redshift/README|Redshift]]** — Petabyte columnar data warehouse and RA3 managed storage.
