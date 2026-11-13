# Amazon EKS (Elastic Kubernetes Service) for Web Development

> **Focus**: Cloud-native web applications on Kubernetes, AWS Load Balancer Controller Ingress, Pod IAM (IRSA), and Karpenter autoscaling.

---

## 1. Kubernetes Web Ingress Architecture with AWS ALB Controller

In production, the **AWS Load Balancer Controller** automatically provisions and configures an AWS Application Load Balancer when developers apply standard Kubernetes `Ingress` manifests.

```mermaid
flowchart TD
    CLIENT["Web Client<br/>(https://api.example.com)"] --> ALB["<b>AWS Application Load Balancer</b><br/>(Managed by AWS LB Controller)"]

    subgraph EKS_CLUSTER["Amazon EKS Cluster (VPC CNI)"]
        direction TB

        INGRESS["Kubernetes Ingress Resource<br/>(alb.ingress.kubernetes.io)"]
        
        subgraph SERVICES["Kubernetes ClusterIP Services"]
            SVC_WEB["Service: web-frontend (:80)"]
            SVC_API["Service: auth-api (:4000)"]
        end

        subgraph PODS["Kubernetes Pod Replicas (Real VPC IPs)"]
            P1["Pod: web-frontend (10.0.2.14)"]
            P2["Pod: web-frontend (10.0.2.29)"]
            P3["Pod: auth-api (10.0.2.77)"]
        end
    end

    ALB -->|Target Type: IP /| P1
    ALB -->|Target Type: IP /| P2
    ALB -->|Target Type: IP /api/auth| P3
    INGRESS -. Configures Rules .-> ALB

    style CLIENT fill:#0284c7,fill-opacity:0.12,stroke:#0284c7,stroke-width:1.5px,color:inherit
    style ALB fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style EKS_CLUSTER fill:#8b5cf6,fill-opacity:0.05,stroke:#8b5cf6,stroke-width:2px,color:inherit
    style PODS fill:#10b981,fill-opacity:0.1,stroke:#10b981,stroke-width:1.5px,color:inherit
```

---

## 2. Pod Security: IAM Roles for Service Accounts (IRSA)

Instead of granting broad IAM permissions to the EC2 worker node, **IRSA** uses OpenID Connect (OIDC) federation to give each web pod its own fine-grained IAM Role.

```mermaid
sequenceDiagram
    autonumber
    participant Pod as Web App Pod
    participant K8s as Kubernetes API Server
    participant STS as AWS STS (Security Token Service)
    participant S3 as Amazon S3 Bucket

    Pod->>K8s: Request ServiceAccount Token (JWT)
    K8s-->>Pod: Injected Projected ServiceAccount Token
    Pod->>STS: AssumeRoleWithWebIdentity(Token, RoleArn)
    STS->>K8s: Validate OIDC Token Signature via OIDC Discovery URL
    STS-->>Pod: Temporary AWS Credentials (Valid 1h)
    Pod->>S3: PutObject / GetObject (Authorized via Pod IAM Role)
```

---

## 3. High-Traffic Web Scaling: HPA + Karpenter

```mermaid
flowchart TD
    TRAFFIC["Incoming Web Traffic Surge"] --> PODS["Pods Under High CPU / Request Load"]
    
    PODS --> HPA["<b>Horizontal Pod Autoscaler (HPA)</b><br/>Detects CPU &gt; 70% -> Scales Pods: 3 -> 20"]
    
    HPA --> SCHED{"Kubernetes Scheduler:<br/>Are Nodes Full?"}
    
    SCHED -->|Pods Pending| KARP["<b>Karpenter Autoscaler</b><br/>Directly launches right-sized EC2 instances in &lt;45s"]
    KARP --> NODES["New Worker Nodes Join Cluster"]
    NODES --> RUNNING["All 20 Web Pods Scheduled & Healthy"]

    style TRAFFIC fill:#ef4444,fill-opacity:0.15,stroke:#ef4444,stroke-width:2px,color:inherit
    style HPA fill:#f59e0b,fill-opacity:0.15,stroke:#f59e0b,stroke-width:2px,color:inherit
    style KARP fill:#0284c7,fill-opacity:0.15,stroke:#0284c7,stroke-width:2px,color:inherit
    style RUNNING fill:#10b981,fill-opacity:0.15,stroke:#10b981,stroke-width:2px,color:inherit
```

---

## 4. Production Kubernetes Web Ingress Manifest

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-app-ingress
  namespace: production
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:123456789012:certificate/abc-123
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS": 443}]'
    alb.ingress.kubernetes.io/ssl-redirect: '443'
spec:
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-api-service
                port:
                  number: 8080
```
