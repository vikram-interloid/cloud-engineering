# Amazon DynamoDB for Web Development

> **Focus**: Single-Table Design for web entities, sub-10ms response times, and optimistic locking for e-commerce carts.

---

## 1. Single-Table Design for Web Applications

Instead of creating separate tables and executing expensive relational SQL `JOIN`s, modern web apps store multiple entities (Users, Orders, Products) in a **single DynamoDB table** using composite partition keys (`PK`) and sort keys (`SK`).

```mermaid
flowchart TD
    subgraph TABLE["Single DynamoDB Table ('WebAppData')"]
        direction TB
        subgraph USER_ITEM["User Entity"]
            U1["PK: USER#101 | SK: METADATA | Name: Alice, Email: alice@example.com"]
        end
        subgraph ORDER_ITEMS["Alice's Orders (1-to-Many Relationship)"]
            O1["PK: USER#101 | SK: ORDER#2024-001 | Total: $49.99, Status: Shipped"]
            O2["PK: USER#101 | SK: ORDER#2024-002 | Total: $129.00, Status: Processing"]
        end
        subgraph GSI_VIEW["Global Secondary Index (GSI1) for Inverse Query"]
            G1["GSI1PK: STATUS#Processing | GSI1SK: 2024-002 | Returns all pending orders"]
        end
    end

    API["Web Backend API"] -->|1 Query returns User Profile AND all Recent Orders in 4ms!| TABLE

    style TABLE fill:#0284c7,fill-opacity:0.06,stroke:#0284c7,stroke-width:2px,color:inherit
    style USER_ITEM fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:1.5px,color:inherit
    style ORDER_ITEMS fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style GSI_VIEW fill:#8b5cf6,fill-opacity:0.15,stroke:#8b5cf6,stroke-width:1.5px,color:inherit
```

---

## 2. Optimistic Locking (Preventing Race Conditions)

When concurrent web requests attempt to modify the same item (e.g., deducting warehouse inventory), DynamoDB **Conditional Writes** ensure data integrity without distributed table locks.

```mermaid
sequenceDiagram
    autonumber
    actor WebUserA as Web User A
    actor WebUserB as Web User B
    participant DDB as Amazon DynamoDB

    WebUserA->>DDB: Read Stock { itemId: "shoe-1", stock: 1, version: 5 }
    WebUserB->>DDB: Read Stock { itemId: "shoe-1", stock: 1, version: 5 }

    WebUserA->>DDB: PutItem: stock=0, version=6 IF version == 5
    DDB-->>WebUserA: 200 OK (Stock updated successfully!)

    WebUserB->>DDB: PutItem: stock=0, version=6 IF version == 5
    DDB-->>WebUserB: 400 ConditionalCheckFailedException!
    Note over WebUserB: App tells user B: 'Sorry, item out of stock!'
```
