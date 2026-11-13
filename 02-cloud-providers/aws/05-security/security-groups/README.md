# Security Groups vs. Network ACLs (NACLs)

> **Focus**: Multi-layered network firewalls: Stateful instance ENI firewalls vs. stateless subnet boundaries.

---

## 1. Network Boundary Comparison Diagram

```mermaid
flowchart LR
    PACKET["Incoming Internet Traffic"] --> NACL["<b>Network ACL (NACL)</b><br/>Subnet Boundary | Stateless | Rule Numbers (100, 200) | Allow & Deny"]
    NACL --> SG["<b>Security Group (SG)</b><br/>Instance ENI Boundary | Stateful | Allow Rules Only"]
    SG --> EC2["EC2 Instance / Container"]

    style PACKET fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style NACL fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:2px,color:inherit
    style SG fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style EC2 fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
```

---

## 2. Security Group Stateful vs NACL Stateless Behavior

```mermaid
flowchart TD
    subgraph SG_FLOW["Security Group (Stateful)"]
        direction TB
        SG_IN["Inbound Port 443 ALLOWED"]
        SG_OUT["Outbound Response Traffic AUTOMATICALLY ALLOWED<br/>(State connection tracked in connection table)"]
        SG_IN --> SG_OUT
    end

    subgraph NACL_FLOW["Network ACL (Stateless)"]
        direction TB
        NACL_IN["Inbound Port 443 ALLOWED"]
        NACL_OUT["Outbound Ephemeral Ports (1024-65535) MUST BE EXPLICITLY ALLOWED!"]
        NACL_IN --> NACL_OUT
    end

    style SG_FLOW fill:#10b981,fill-opacity:0.1,stroke:#10b981,stroke-width:2px,color:inherit
    style NACL_FLOW fill:#ef4444,fill-opacity:0.1,stroke:#ef4444,stroke-width:2px,color:inherit
```

---

## 3. Comparison Matrix

| Property | Security Group | Network ACL (NACL) |
| :--- | :--- | :--- |
| **Operates At** | Instance / ENI level | Subnet level |
| **Rules Supported**| Allow rules only | Allow AND Deny rules |
| **State** | **Stateful** (Return traffic automatically permitted) | **Stateless** (Return traffic requires explicit rule) |
| **Rule Order** | All rules evaluated before decision | Evaluated in sequential rule order (lowest # first)|
