# Module 01 · Cloud Computing Fundamentals & Architecture

> **Path**: `00-Foundations/01-cloud-computing/`  
> **Master Overview**: [[Overview\|Overview & Master Diagram]]  
> **Status**: Complete & Verified

---

## Module Overview

This module covers the core foundational principles, historical evolution, architectural components, service models, and economic frameworks of modern cloud computing.

```
00-Foundations/01-cloud-computing/
├── README.md                                          (This Index)
├── 01_What_is_Cloud_Computing.md                      (NIST definition, utility model, control plane)
├── 02_Evolution_of_Computing_OnPrem_to_Cloud.md       (Mainframes -> Client-Server -> Virtualization -> Cloud)
├── 03_Traditional_Data_Center_Architecture.md         (Physical DC, 3-tier networking, SAN, PUE)
├── 04_Why_Cloud_Computing.md                          (Agility, elasticity, global reach, anti-patterns)
├── 05_Cloud_Computing_Characteristics.md              (NIST 5 essential characteristics + extended)
├── 06_Cloud_Service_Models_IaaS_PaaS_SaaS.md          (IaaS, PaaS, FaaS, SaaS comparison spectrum)
├── 07_Cloud_Deployment_Models.md                      (Public, Private, Hybrid, Multi-Cloud, Edge)
├── 08_Cloud_Infrastructure_Components.md              (Regions, AZs, SDN, VPC, Compute, Storage, DBs)
├── 09_Cloud_Shared_Responsibility_Model.md            (Security of the cloud vs in the cloud matrix)
└── 10_Cloud_Economics_CapEx_vs_OpEx.md                (TCO iceberg, FinOps, On-Demand/Savings/Spot)
```

---

## Detailed Curriculum Index

1. [[01_What_is_Cloud_Computing|01 · What is Cloud Computing?]]
   - The NIST SP 800-145 formal standard
   - Programmable API control plane vs. high-throughput data plane
   - True cloud vs. hosted virtualization
2. [[02_Evolution_of_Computing_OnPrem_to_Cloud|02 · Evolution of Computing – On-Premises to Cloud]]
   - Historical epochs: Mainframe, Client-Server, 3-Tier Web, Type-1 Hypervisors
   - Containers, Microservices, and Serverless event execution
3. [[03_Traditional_Data_Center_Architecture|03 · Traditional Data Center Architecture]]
   - Facility design (UPS, ATS, dual-path power, CRAC chillers, Tier I-IV)
   - Core-Aggregation-Access tree vs. Cloud Spine-Leaf (Clos) networking
   - Storage topologies: Direct Attached (DAS), NAS, and Fibre Channel SAN
4. [[04_Why_Cloud_Computing|04 · Why Cloud Computing?]]
   - Eliminating undifferentiated heavy lifting
   - Velocity, dynamic capacity matching, high availability by default
   - Architectural trade-offs and when NOT to choose the cloud
5. [[05_Cloud_Computing_Characteristics|05 · Cloud Computing Characteristics]]
   - The 5 essential NIST characteristics: On-demand self-service, Broad network access, Resource pooling, Rapid elasticity, Measured service
   - Modern extended characteristics: Resiliency, Declarative GitOps, Fine-grained IAM
6. [[06_Cloud_Service_Models_IaaS_PaaS_SaaS|06 · Cloud Service Models – IaaS, PaaS, SaaS & FaaS]]
   - Abstraction spectrum: Maximum control to maximum convenience
   - In-depth technical breakdown and comparison matrix
7. [[07_Cloud_Deployment_Models|07 · Cloud Deployment Models]]
   - Public, Private, Hybrid (DirectConnect/ExpressRoute), and Multi-Cloud
   - Edge and distributed cloud (AWS Outposts, Google Anthos, Wavelength)
8. [[08_Cloud_Infrastructure_Components|08 · Cloud Infrastructure Components]]
   - Regions, Availability Zones (sub-2ms), and Edge Points of Presence
   - Software-Defined Networking (VPC, Subnets, NAT, ALB, Security Groups)
   - The 8 core building blocks with cross-cloud mapping (AWS, Azure, GCP)
9. [[09_Cloud_Shared_Responsibility_Model|09 · Cloud Shared Responsibility Model]]
   - Security "OF" the cloud vs. Security "IN" the cloud
   - Stacked accountability matrix across IaaS, PaaS, and SaaS
   - Real-world cloud security breaches and misconfigurations
10. [[10_Cloud_Economics_CapEx_vs_OpEx|10 · Cloud Economics – CapEx vs. OpEx]]
    - Capital vs. operational expense accounting
    - Total Cost of Ownership (TCO) iceberg analysis
    - Purchasing models: On-Demand, Savings Plans, and Spot instances
    - The FinOps lifecycle and cloud unit economics

---

## Visual Architecture Blueprint

View the master interactive infographic in [[Overview|Overview]] or open [cloud_architecture_service_models.html](file:///home/vikram/learning/cloud/Cloud_Learning/Overview/cloud_architecture_service_models.html) directly in any web browser.
