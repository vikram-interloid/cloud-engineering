# Amazon ElastiCache

> **Focus**: In-memory data store for Redis and Memcached, sub-millisecond caching, and cache strategies.

---

## 1. Cache-Aside (Lazy Loading) Architecture Pattern

```mermaid
sequenceDiagram
    autonumber
    actor App as App Server
    participant Cache as Amazon ElastiCache (Redis)
    participant DB as Amazon RDS / Aurora

    App->>Cache: 1. GET user:10024
    alt Cache Hit (Microseconds)
        Cache-->>App: Return User Data (Hit)
    else Cache Miss
        Cache-->>App: Null (Miss)
        App->>DB: 2. SELECT * FROM users WHERE id=10024
        DB-->>App: Return User Record
        App->>Cache: 3. SET user:10024 (TTL = 3600s)
    end
```

---

## 2. Redis vs. Memcached Decision Tree

```mermaid
flowchart TD
    DEC{"In-Memory Store Selection"}
    
    DEC -->|Data persistence, Multi-AZ failover, Pub/Sub, Geospatial, Sorted Sets| REDIS["<b>ElastiCache for Redis</b><br/>Complex data structures, clustering, read replicas, encryption"]
    DEC -->|Pure simple object caching, multi-threaded horizontal scaling| MEM["<b>ElastiCache for Memcached</b><br/>Simple key-value, multi-core CPU scaling, no persistence"]

    style DEC fill:#64748b,fill-opacity:0.15,stroke:#64748b,stroke-width:1.5px,color:inherit
    style REDIS fill:#ef4444,fill-opacity:0.15,stroke:#ef4444,stroke-width:2px,color:inherit
    style MEM fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
```
