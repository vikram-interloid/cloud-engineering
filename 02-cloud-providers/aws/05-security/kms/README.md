# AWS KMS (Key Management Service)

> **Focus**: Cryptographic key hierarchies, HSM-backed storage, and envelope encryption.

---

## 1. Envelope Encryption Architecture

AWS KMS uses **Envelope Encryption** to encrypt data: plaintext data is encrypted with a unique **Data Key**, and the Data Key itself is encrypted under a KMS **KMS Key (CMK)**.

```mermaid
sequenceDiagram
    autonumber
    actor Client as Application Service
    participant KMS as AWS KMS (HSM Core)
    participant Storage as Amazon S3 / EBS Disk

    Note over Client,KMS: ENCRYPT DATA FLOW
    Client->>KMS: GenerateDataKey(KeyId="alias/my-key")
    KMS-->>Client: Plaintext Data Key + Encrypted Data Key (Ciphertext)
    Client->>Client: Encrypt Data with Plaintext Data Key (AES-GCM-256)
    Client->>Client: Erase Plaintext Data Key from Memory!
    Client->>Storage: Store Encrypted Data + Encrypted Data Key

    Note over Client,KMS: DECRYPT DATA FLOW
    Storage-->>Client: Fetch Encrypted Data + Encrypted Data Key
    Client->>KMS: Decrypt(EncryptedDataKey)
    KMS-->>Client: Plaintext Data Key
    Client->>Client: Decrypt Data & Discard Key from RAM
```

---

## 2. KMS Key Types Comparison

| Key Type | Managed By | Key Rotation | Cross-Account Sharing |
| :--- | :--- | :--- | :--- |
| **AWS Owned Keys** | AWS Service | Internal AWS schedule | No (Free internal default) |
| **AWS Managed Keys** (`aws/s3`) | AWS Service for your account | Automatic every 1 year | No |
| **Customer Managed (CMK)** | Customer (Full Control) | Automatic yearly or on-demand | Yes (via Key Policies) |
