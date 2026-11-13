# AWS Cloud Development Kit (CDK)

> **Focus**: Defining cloud infrastructure using familiar programming languages (TypeScript, Python, Go).

---

## 1. CDK Synthesis & Deployment Pipeline

```mermaid
flowchart TD
    CODE["CDK Code (TypeScript / Python)<br/>Defines Apps, Stacks & Constructs"] --> SYNTH["<b>cdk synth</b><br/>Synthesizes code into CloudFormation template"]
    SYNTH --> TEMPLATE["CloudFormation JSON / YAML Template + Assets"]
    TEMPLATE --> BOOTSTRAP["CDK Toolkit S3 Bucket (Assets Cache)"]
    TEMPLATE --> DEPLOY["<b>cdk deploy</b><br/>CloudFormation Engine provisions resources"]
    DEPLOY --> AWS_RESOURCES["AWS Resources Created (VPC, ECS, DynamoDB)"]

    style CODE fill:#3b82f6,fill-opacity:0.15,stroke:#3b82f6,stroke-width:2px,color:inherit
    style SYNTH fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:1.5px,color:inherit
    style DEPLOY fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
    style AWS_RESOURCES fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:1.5px,color:inherit
```

---

## 2. CDK Construct Abstraction Levels

```mermaid
flowchart LR
    L1["<b>L1 Constructs (CfnResource)</b><br/>1:1 direct CloudFormation mapping<br/>(e.g., CfnBucket)"]
    L2["<b>L2 Constructs (AWS Curated)</b><br/>Sensible defaults, built-in security<br/>(e.g., s3.Bucket)"]
    L3["<b>L3 Constructs (Solutions)</b><br/>Full multi-service architecture patterns<br/>(e.g., ApplicationLoadBalancedFargateService)"]

    L1 --> L2 --> L3

    style L1 fill:#64748b,fill-opacity:0.15,stroke:#64748b,stroke-width:1.5px,color:inherit
    style L2 fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style L3 fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
```
