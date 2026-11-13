# AWS WAF for Web Development

> **Focus**: Protecting web applications against OWASP Top 10 vulnerabilities, login brute-force attacks, and scraping bots.

---

## 1. Web Application Firewall Defense Flow

AWS WAF inspects HTTP/HTTPS headers, cookies, query parameters, and JSON payloads at the edge **before** traffic reaches application servers.

```mermaid
flowchart TD
    REQUEST["Incoming Web Traffic"] --> WAF["<b>AWS WAF Web ACL</b>"]

    subgraph INSPECTION["Web Security Inspection Hierarchy"]
        IP_REP["1. IP Reputation & Geoblocking (Block malicious proxies)"]
        RATE["2. Rate-Limiting Rule (Block &gt;100 requests / 5m per IP on /api/login)"]
        CRS["3. AWS Managed Core Rule Set (Mitigate SQLi, XSS, Path Traversal)"]
        BOT["4. AWS Bot Control (Challenge scrapers & headless crawlers)"]
        IP_REP --> RATE --> CRS --> BOT
    end

    WAF --> INSPECTION
    INSPECTION -->|Violation Detected| BLOCK["403 Forbidden / CAPTCHA Challenge"]
    INSPECTION -->|Clean Request| ORIGIN["Forward to CloudFront / ALB / API Gateway"]

    style REQUEST fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style WAF fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style BLOCK fill:#ef4444,fill-opacity:0.2,stroke:#ef4444,stroke-width:2px,color:inherit
    style ORIGIN fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
```

---

## 2. Essential Web Developer WAF Rules

| Protection | Implementation Rule | Targeted Threat |
| :--- | :--- | :--- |
| **Login Brute-Force** | Rate-based rule: Limit to 50 requests per 5 min on URI `/api/auth/login` | Credential stuffing & password guessing |
| **SQL Injection** | `AWSManagedRulesSQLiRuleSet` | Database compromise via query tampering |
| **Cross-Site Scripting** | `AWSManagedRulesCommonRuleSet` (SizeRestrictions_BODY & CrossSiteScripting_BODY) | Malicious script execution in user browsers |
| **Scraper Bots** | AWS Bot Control Managed Rule Set | Automated price scraping & content theft |
