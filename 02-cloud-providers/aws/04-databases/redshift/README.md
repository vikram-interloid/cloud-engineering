# Amazon Redshift

> **Focus**: Petabyte-scale columnar data warehouse, Massively Parallel Processing (MPP), and RA3 storage.

---

## 1. Redshift MPP Cluster Architecture

```mermaid
flowchart TD
    SQL["BI Tools / SQL Client"] --> LEADER["<b>Leader Node</b><br/>Parses SQL, builds execution plan, aggregates results"]
    
    subgraph CLUSTER["Compute Node Fleet (Massively Parallel Processing)"]
        direction LR
        subgraph N1["Compute Node 1"]
            S1["Slice 1"]
            S2["Slice 2"]
        end
        subgraph N2["Compute Node 2"]
            S3["Slice 3"]
            S4["Slice 4"]
        end
    end

    subgraph STORAGE["Amazon Redshift Managed Storage (RMS)"]
        S3_BACKED["S3-Backed Storage Fabric (Automatic SSD Caching)"]
    end

    LEADER --> N1
    LEADER --> N2
    N1 <== High Bandwidth ==> STORAGE
    N2 <== High Bandwidth ==> STORAGE

    style LEADER fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style CLUSTER fill:#10b981,fill-opacity:0.08,stroke:#10b981,stroke-width:1.5px,color:inherit
    style STORAGE fill:#f59e0b,fill-opacity:0.1,stroke:#f59e0b,stroke-width:1.5px,color:inherit
```

---

## 2. Row vs. Columnar Storage (Why Redshift is Fast for Analytics)

```mermaid
flowchart LR
    subgraph ROW_STORE["Traditional Row Store (Reads entire row for sum)"]
        direction TB
        R1["Row 1: [ID, Name, Country, Revenue]"]
        R2["Row 2: [ID, Name, Country, Revenue]"]
        R3["Row 3: [ID, Name, Country, Revenue]"]
    end

    subgraph COL_STORE["Redshift Columnar Store (Reads only Revenue column)"]
        direction TB
        C1["Revenue: [100.50, 450.00, 29.99...]"]
        C2["Scans 1/10th of disk I/O with high compression"]
    end

    style ROW_STORE fill:#ef4444,fill-opacity:0.1,stroke:#ef4444,stroke-width:1.5px,color:inherit
    style COL_STORE fill:#10b981,fill-opacity:0.1,stroke:#10b981,stroke-width:1.5px,color:inherit
```
