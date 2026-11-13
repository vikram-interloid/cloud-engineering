# Module 05 · Networking Fundamentals for Cloud Engineers

> **Path**: `00-Foundations/05-networking-fundamentals/`  
> **Master Overview**: [[Overview|Overview]] | [[00-Foundations/01-cloud-computing/README|Cloud Computing Index]]  
> **Status**: Complete & Verified

---

## Module Overview

Networking is the nervous system of cloud computing. Every packet, API invocation, container microservice, and database query depends directly on reliable, secure, and low-latency network primitives.

This foundational module provides a rigorous, in-depth reference for computer networking principles—spanning from physical layer signaling and the OSI model up to software-defined overlay fabrics, modern HTTP/3, and hardware SmartNIC acceleration.

```
00-Foundations/05-networking-fundamentals/
├── README.md                                          (This Index)
├── 01_OSI_and_TCP_IP_Models.md                        (7-layer OSI vs 4-layer TCP/IP, encapsulation, PDUs)
├── 02_IP_Addressing_and_Subnetting.md                 (IPv4/IPv6, binary subnetting, CIDR notation, RFC 1918)
├── 03_Routing_and_Switching.md                        (CAM tables, ARP, VLANs, Longest Prefix Match, BGP)
├── 04_Core_Network_Protocols.md                       (DNS hierarchy, record types, DHCP DORA, ICMP PMTUD)
├── 05_Transport_Protocols_TCP_UDP.md                  (TCP 3-way handshake, teardown, sliding window, QUIC)
├── 06_Application_Protocols_HTTP_HTTPS_TLS.md         (HTTP/1.1 to HTTP/3, TLS 1.3 handshake, PKI, mTLS)
├── 07_Network_Security_Firewalls_and_NAT.md           (Stateful vs stateless, SNAT, PAT, DMZ, Security Groups)
└── 08_Cloud_Software_Defined_Networking_SDN.md        (SDN planes, underlay vs overlay, VXLAN, Geneve, Nitro)
```

---

## Detailed Curriculum Index

1. [[01_OSI_and_TCP_IP_Models|01 · OSI and TCP/IP Networking Models]]
   - The theoretical 7-layer OSI model vs. the pragmatic 4-layer TCP/IP stack.
   - Protocol Data Units (PDUs): Bits, Frames, Packets, Segments, and Application Data.
   - Downward encapsulation and upward decapsulation workflows.
2. [[02_IP_Addressing_and_Subnetting|02 · IP Addressing, Subnetting, and CIDR Notation]]
   - 32-bit IPv4 structures, binary masks, and network vs. host calculations.
   - Classless Inter-Domain Routing (CIDR) reference matrices.
   - Cloud-reserved IP addresses (AWS 5-IP per subnet reservation).
   - RFC 1918 private address spaces and IPv6 128-bit architecture.
3. [[03_Routing_and_Switching|03 · Routing and Switching – Layer 2 vs. Layer 3]]
   - Layer 2 MAC switching, Content Addressable Memory (CAM) tables.
   - Address Resolution Protocol (ARP) mechanics and cloud hypervisor virtualization.
   - Longest Prefix Match (LPM) routing rules and Border Gateway Protocol (BGP).
   - Physical hardware routers vs. software-defined cloud VPC route tables.
4. [[04_Core_Network_Protocols|04 · Core Network Protocols – DNS, DHCP, and ICMP]]
   - Distributed DNS tree: Root, TLD, Authoritative, and Recursive Resolvers.
   - Essential DNS records (`A`, `AAAA`, `CNAME`, `ALIAS`, `MX`, `TXT`).
   - DHCP four-way DORA lease lifecycle (Discover, Offer, Request, Acknowledge).
   - ICMP error signaling, `traceroute` TTL diagnostics, and Path MTU Discovery (PMTUD).
5. [[05_Transport_Protocols_TCP_UDP|05 · Transport Protocols – TCP, UDP, and QUIC]]
   - Layer 4 socket multiplexing and port allocation ranges.
   - TCP 3-way handshake, 4-way connection teardown, and `TIME_WAIT` safety timers.
   - Sliding window flow control (`rwnd`) vs. network congestion control (`cwnd`).
   - UDP characteristics and QUIC (HTTP/3) connection migration over UDP.
6. [[06_Application_Protocols_HTTP_HTTPS_TLS|06 · Application Protocols – HTTP/1.1, HTTP/2, HTTP/3, and TLS 1.3]]
   - HTTP evolution from serial pipelining to multiplexed binary framing.
   - REST verbs, idempotency, and standard HTTP status code taxonomy.
   - TLS 1.3 1-RTT cryptographic handshakes and Perfect Forward Secrecy (PFS).
   - X.509 PKI certificate chains of trust and Mutual TLS (mTLS) for microservices.
7. [[07_Network_Security_Firewalls_and_NAT|07 · Network Security – Firewalls, NAT, and DMZ Architecture]]
   - Firewall taxonomies: Stateless packet filters, stateful inspection, and WAFs.
   - Stateful Security Groups vs. Stateless Network ACLs.
   - Source NAT (SNAT), Destination NAT (DNAT), and Port Address Translation (PAT).
   - Demilitarized Zone (DMZ) multi-tier application isolation topologies.
8. [[08_Cloud_Software_Defined_Networking_SDN|08 · Cloud Software-Defined Networking (SDN) and Overlay Networks]]
   - Decoupling the centralized Control Plane from distributed Data Planes.
   - Physical underlay fabric vs. customer virtual overlay networks.
   - VXLAN (RFC 7348) and Geneve (RFC 8926) packet encapsulation formats.
   - Hardware SmartNIC offloading (AWS Nitro, Azure FPGA) eliminating virtualization overhead.

---

## Next Steps

> ➡️ **Proceed to Module 06**: [[00-Foundations/06-compute-fundamentals/README|06 · Compute Fundamentals]]  
> *Dive deep into CPU microarchitectures, RAM hierarchies, OS kernels, process scheduling, Type-1 hypervisors, Linux cgroups/namespaces, and serverless microVM sandboxes.*
