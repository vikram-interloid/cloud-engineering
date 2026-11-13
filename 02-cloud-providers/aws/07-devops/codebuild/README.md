# AWS CodeBuild

> **Focus**: Serverless build and continuous integration service, custom Docker runtimes, and buildspec lifecycle.

---

## 1. Build Lifecycle Phases (`buildspec.yml`)

```mermaid
flowchart TD
    START["CodeBuild Container Booted"] --> P1["<b>install</b><br/>Install runtimes, packages, tools (e.g. npm install, pip)"]
    P1 --> P2["<b>pre_build</b><br/>Login to Amazon ECR, check prerequisites, linters"]
    P2 --> P3["<b>build</b><br/>Run unit tests, compile binaries, build Docker image"]
    P3 --> P4["<b>post_build</b><br/>Tag & push Docker image to ECR, produce artifacts"]
    P4 --> FINISH["Export Build Artifacts to S3 & Terminate Container"]

    style START fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style P1 fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style P2 fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style P3 fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style P4 fill:#8b5cf6,fill-opacity:0.15,stroke:#8b5cf6,stroke-width:1.5px,color:inherit
```

---

## 2. Key Operational Features

- **Elastic Scaling**: Spawns parallel dedicated containers per build; zero build queue contention.
- **Pay-Per-Minute**: Billed only for compute duration consumed during build phases.
