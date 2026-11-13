# Lab 01 · Build a Production Multi-AZ VPC

> **Objective**: Provision a 2-AZ VPC containing Public & Private Subnets, an Internet Gateway, a NAT Gateway, and dedicated Route Tables.

---

## Target Architecture

```mermaid
flowchart TD
    INTERNET["Public Internet"]

    subgraph VPC["Production VPC: 10.0.0.0/16"]
        IGW["Internet Gateway"]
        INTERNET <==> IGW

        subgraph AZ_A["Availability Zone us-east-1a"]
            PUB_A["Public Subnet A<br/>10.0.1.0/24"]
            NAT_A["NAT Gateway A"]
            PRIV_A["Private Subnet A<br/>10.0.2.0/24"]
            PUB_A --- NAT_A
            NAT_A -.-> PRIV_A
        end

        subgraph AZ_B["Availability Zone us-east-1b"]
            PUB_B["Public Subnet B<br/>10.0.11.0/24"]
            PRIV_B["Private Subnet B<br/>10.0.12.0/24"]
        end

        IGW <--> PUB_A
        IGW <--> PUB_B
    end

    style VPC fill:#0284c7,fill-opacity:0.05,stroke:#0284c7,stroke-width:2px,color:inherit
    style AZ_A fill:#10b981,fill-opacity:0.08,stroke:#10b981,stroke-width:1.5px,color:inherit
    style AZ_B fill:#f59e0b,fill-opacity:0.08,stroke:#f59e0b,stroke-width:1.5px,color:inherit
```

---

## Execution Steps

```bash
# 1. Create VPC
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query 'Vpc.VpcId' --output text)

# 2. Create Internet Gateway & Attach
IGW_ID=$(aws ec2 create-internet-gateway --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $IGW_ID

# 3. Create Public Subnet in us-east-1a
PUB_SUB=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.1.0/24 --availability-zone us-east-1a --query 'Subnet.SubnetId' --output text)

# 4. Create Public Route Table & Default Route
RT_PUB=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --route-table-id $RT_PUB --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID
aws ec2 associate-route-table --subnet-id $PUB_SUB --route-table-id $RT_PUB
```
