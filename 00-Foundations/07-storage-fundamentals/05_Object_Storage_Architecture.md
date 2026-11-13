# 05 · Object Storage Architecture – Flat Namespaces, Buckets, and REST APIs

> **Status**: Comprehensive Reference  
> **Parent**: [[Overview|Overview]] | [[00-Foundations/07-storage-fundamentals/README|Storage Index]]  
> **Topics**: Object Storage Philosophy, Buckets & Flat Namespaces, REST API Primitives, Multipart Upload Mechanics, Strong Read-After-Write Consistency, WORM Compliance (Object Lock)

---

## 1. The Object Storage Paradigm

Traditional filesystems collapse when scaled to billions of files because directory tree traversals, file locking mechanisms, and inode lookups create severe disk I/O bottlenecks.

**Cloud Object Storage** abandons hierarchical directories entirely in favor of an **infinitely scalable, flat key-value store** accessible globally via standard **HTTP/HTTPS REST APIs**.

```mermaid
flowchart LR
    subgraph REST_Client["Client Application / Browser"]
        direction TB
        Client["Application (cURL, AWS SDK, Python boto3)"]
    end

    subgraph API_Gateway["Object Storage Front-End Fleet"]
        direction TB
        LoadBalancer["Global Anycast Load Balancer"]
        REST_Engines["HTTP REST Engine Fleet (Parses GET, PUT, DELETE, HEAD)"]
        LoadBalancer --> REST_Engines
    end

    subgraph MetadataPlane["Distributed Metadata Engine (Consensus Cluster)"]
        direction TB
        MetaDB["High-Speed Key-Value Index (Raft / Paxos Consensus)<br/>Maps: <code>s3://bucket/key</code> -> [Shard Locations & Version IDs]"]
    end

    subgraph StoragePools["Commodity Storage Fleet (Erasure Coded Slices)"]
        direction LR
        S1["Storage Node 1"]
        S2["Storage Node 2"]
        S3["Storage Node 3"]
        S4["Storage Node 4"]
    end

    Client ==>|HTTPS Port 443| LoadBalancer
    REST_Engines ==>|Metadata & Auth Verification| MetaDB
    REST_Engines ==>|Parallel Chunk Streams| StoragePools

    style REST_Client fill:#0284c7,stroke:#0284c7,stroke-width:1.5px,fill-opacity:0.08
    style API_Gateway fill:#8b5cf6,stroke:#8b5cf6,stroke-width:1.5px,fill-opacity:0.08
    style MetadataPlane fill:#f59e0b,stroke:#f59e0b,stroke-width:1.5px,fill-opacity:0.08
    style StoragePools fill:#10b981,stroke:#10b981,stroke-width:1.5px,fill-opacity:0.08

    classDef nodeStyle fill:#0f172a,stroke:#38bdf8,stroke-width:1px,fill-opacity:0.15,color:inherit;
    class Client,LoadBalancer,REST_Engines,MetaDB,S1,S2,S3,S4 nodeStyle;
    linkStyle default stroke:#64748b,stroke-width:2px;
```

---

## 2. Core REST API Primitives

Every interaction with an object storage system maps directly to standard HTTP methods:

| HTTP Verb | S3 / Blob API Action | Operational Function |
| :--- | :--- | :--- |
| **`PUT`** | `PutObject` | Uploads a new object or completely overwrites an existing object with the supplied payload. |
| **`GET`** | `GetObject` | Downloads the full binary payload of the object. Supports **Byte-Range Requests** (`Range: bytes=0-1048575`). |
| **`HEAD`** | `HeadObject` | Retrieves object metadata headers (Content-Length, ETag, Last-Modified) **without transferring payload bytes**. |
| **`DELETE`**| `DeleteObject`| Removes the specified object key (or inserts an immutable Delete Marker if versioning is active). |
| **`POST`** | `CompleteMultipartUpload`| Coordinates complex multi-chunk ingestion workflows. |

---

## 3. High-Throughput Ingestion: Multipart Uploads

Attempting to upload a single 50 GB video or database backup over a standard HTTP `PUT` connection is brittle: a single network hiccup forces the application to restart the upload from byte zero.

Object storage solves this via **Multipart Upload**:

```mermaid
sequenceDiagram
    autonumber
    participant App as Client Application
    participant S3 as Amazon S3 / Azure Blob
    participant Storage as Distributed Storage Fleet

    App->>S3: 1. InitiateMultipartUpload(Bucket, Key)<br/>"Prepare to receive large object"
    S3-->>App: Returns unique UploadId: "abc-123-upload-token"

    par Parallel Part Uploads (Concurrently over separate TCP streams)
        App->>S3: UploadPart(Part 1, 100MB, UploadId)
        S3->>Storage: Writes Chunk 1 & returns ETag1
        App->>S3: UploadPart(Part 2, 100MB, UploadId)
        S3->>Storage: Writes Chunk 2 & returns ETag2
        App->>S3: UploadPart(Part 3, 100MB, UploadId)
        S3->>Storage: Writes Chunk 3 & returns ETag3
    end

    App->>S3: 2. CompleteMultipartUpload(UploadId, [ETag1, ETag2, ETag3])
    S3->>Storage: Atomically links chunks into unified object
    S3-->>App: HTTP 200 OK (Object fully created and available)
```

### Key Advantages of Multipart Upload
1. **Parallelism**: Ingest chunks concurrently, saturating 10Gbps or 100Gbps network interfaces.
2. **Fault Isolation**: If Part 2 fails due to a network timeout, **only Part 2 is retransmitted**, rather than the entire 50 GB file.
3. **Mandatory Standard**: Recommended for objects larger than 100 MB; required for any object exceeding 5 GB.

---

## 4. The Data Consistency Revolution: Eventual to Strong Consistency

Historically (prior to December 2020), Amazon S3 operated under an **Eventual Consistency** model to prioritize high availability during distributed network partitions (CAP Theorem):
- Creating a new object provided immediate read-after-write consistency.
- However, **updating or deleting an existing object** exhibited eventual consistency: reading an updated key might return the old data for several seconds while replica synchronization settled.

### Modern Cloud Object Storage: Strong Read-After-Write Consistency
Modern cloud hyperscalers (Amazon S3, Azure Blob Storage, Google Cloud Storage) now deliver **Strong Read-After-Write Consistency for ALL operations (PUT, UPDATE, DELETE)** with zero performance penalty:
- When a client receives an `HTTP 200 OK` response to a `PUT` or `DELETE`, **any subsequent `GET` or `LIST` request immediately sees the update**.
- Achieved by integrating distributed consensus protocols (Raft/Paxos) into the metadata indexing plane, ensuring that metadata commits synchronously before the HTTP response is sent.

```
┌────────────────────────────────────────────────────────────────────────┐
│               STRONG READ-AFTER-WRITE CONSISTENCY GUARANTEE            │
├────────────────────────────────────────────────────────────────────────┤
│ 1. Client uploads:   PUT s3://bucket/config.json (Payload: {"v": 2})   │
│ 2. S3 responds:      HTTP 200 OK                                       │
│ 3. Client executes:  GET s3://bucket/config.json                      │
│ 4. Result:           GUARANTEED to return {"v": 2} (NEVER {"v": 1})    │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Regulatory Compliance & Immutability: WORM and Object Lock

Certain regulated industries (financial services SEC Rule 17a-4, healthcare HIPAA) mandate that audit logs and customer records be stored in a **Write Once, Read Many (WORM)** format that cannot be deleted or overwritten by anyone—not even the AWS root account holder.

```mermaid
flowchart TD
    subgraph BucketLock["S3 Object Lock Architecture (WORM Compliance)"]
        direction TB

        subgraph LegalHold["1. Legal Hold"]
            LH_D["Can be placed or removed by authorized compliance officers at any time<br/>Indefinite retention; prevents deletion until explicitly cleared"]
        end

        subgraph RetentionModes["2. Retention Period Modes"]
            direction LR
            subgraph GovMode["Governance Mode"]
                G_D["Protects against accidental deletion<br/>Users with special IAM permissions can bypass or shorten period"]
            end
            subgraph CompMode["Compliance Mode (Strict WORM)"]
                C_D["<b>ABSOLUTE IMMUTABILITY</b><br/>Cannot be deleted, overwritten, or shortened by <b>ANYONE</b> (including AWS Root Account)<br/>Guaranteed by SEC 17a-4 compliance"]
            end
        end

        LegalHold --- RetentionModes
    end

    style BucketLock fill:#0284c7,stroke:#0284c7,stroke-width:1.5px,fill-opacity:0.08
    style LegalHold fill:#8b5cf6,stroke:#8b5cf6,stroke-width:1px,fill-opacity:0.08
    style GovMode fill:#f59e0b,stroke:#f59e0b,stroke-width:1px,fill-opacity:0.08
    style CompMode fill:#ef4444,stroke:#ef4444,stroke-width:1px,fill-opacity:0.08

    classDef nodeStyle fill:#0f172a,stroke:#38bdf8,stroke-width:1px,fill-opacity:0.15,color:inherit;
    class LH_D,G_D,C_D nodeStyle;
    linkStyle default stroke:#64748b,stroke-width:2px;
```

---

## 6. Key Takeaways

| Object Storage Dimension | Core Architectural Principle |
| :--- | :--- |
| **Flat Namespace Scalability**| Abandoning hierarchical directories allows object storage to scale to trillions of files without inode or path bottlenecks. |
| **Multipart Ingestion Speed**| Multipart uploads split large files into parallel chunks, providing resilience against network drops and maximizing bandwidth. |
| **Strong Consistency Modernity**| Cloud object stores now guarantee strong read-after-write consistency across all PUTs, updates, and deletes. |
| **WORM Immutability** | Compliance-mode Object Lock ensures stored data cannot be destroyed by ransomware or rogue root administrators. |

---

## 7. Knowledge Check

1. Why does an object storage system scale to trillions of items far more easily than a traditional hierarchical filesystem?
2. What are the four core HTTP methods used to interact with cloud object storage?
3. What is the operational purpose of an `HTTP HEAD` request in object storage?
4. How do Byte-Range Requests (`Range: bytes=X-Y`) enable video streaming platforms like Netflix or YouTube to stream video efficiently from S3?
5. Walk through the three phases of an S3 Multipart Upload.
6. What is the minimum recommended file size for initiating a Multipart Upload?
7. Explain the difference between legacy Eventual Consistency and modern Strong Read-After-Write Consistency in Amazon S3.
8. How does distributed consensus (e.g., Paxos or Raft) enable strong consistency across geographically replicated metadata clusters?
9. What is the difference between Governance Mode and Compliance Mode in S3 Object Lock?
10. Can an AWS root account user delete a file locked under S3 Object Lock in Compliance Mode during its retention window?

---

## 8. Next Chapter

Continue your storage fundamentals learning journey:

> ➡️ **Next Module**: [[06_Storage_Tiering_Lifecycle_and_Backup|06 · Storage Tiering, Lifecycle Management, and Disaster Recovery]]  
> *Exploring Hot vs. Cool vs. Archive storage economics, automated lifecycle transitions, snapshot copy-on-write mechanics, Recovery Point Objective (RPO), and Recovery Time Objective (RTO).*

---

### Authoritative References

- [Amazon Web Services: Amazon S3 Architecture and Performance Guidelines](https://docs.aws.amazon.com/AmazonS3/latest/userguide/optimizing-performance.html)
- [Amazon Web Services: Amazon S3 Strong Consistency Announcement](https://aws.amazon.com/blogs/aws/amazon-s3-update-strong-read-after-write-consistency/)
- [Microsoft Azure: Azure Blob Storage Architecture Guide](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction)
