# 05 · AWS Security, Identity & Compliance

> **Module**: Security  
> **Focus**: Defense-in-depth, identity federation, cryptographic key management, secrets rotation, and perimeter firewalls.

---

## Defense-in-Depth Security Onion

```mermaid
flowchart TD
    EDGE["1. Edge & Perimeter Defense (AWS Shield, WAF, CloudFront)"]
    NET["2. Network Security (VPC, Subnets, Security Groups, NACLs)"]
    IAM["3. Identity & Access Management (Least Privilege, MFA, Roles)"]
    COMPUTE["4. Compute & Host Hardening (Systems Manager, GuardDuty, Inspector)"]
    DATA["5. Data Protection & Cryptography (KMS Envelope Encryption, Macie)"]

    EDGE --> NET --> IAM --> COMPUTE --> DATA

    style EDGE fill:#ef4444,fill-opacity:0.12,stroke:#ef4444,stroke-width:2px,color:inherit
    style NET fill:#f59e0b,fill-opacity:0.12,stroke:#f59e0b,stroke-width:2px,color:inherit
    style IAM fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:2px,color:inherit
    style COMPUTE fill:#8b5cf6,fill-opacity:0.12,stroke:#8b5cf6,stroke-width:2px,color:inherit
    style DATA fill:#10b981,fill-opacity:0.12,stroke:#10b981,stroke-width:2px,color:inherit
```

---

## Security Directory Index

- **[[iam/README|IAM]]** — Identity and Access Management, policy evaluation logic, and role delegation.
- **[[kms/README|KMS]]** — Key Management Service and cryptographic envelope encryption.
- **[[secrets-manager/README|Secrets Manager]]** — Automated credential storage and Lambda lifecycle rotation.
- **[[security-groups/README|Security Groups]]** — Stateful instance-level firewalls vs. stateless NACLs.
- **[[waf/README|WAF]]** — Web Application Firewall, rule sets, and OWASP Top 10 mitigation.
