# Module 07 · Storage Fundamentals for Cloud Engineers

> **Path**: `00-Foundations/07-storage-fundamentals/`  
> **Master Overview**: [[Overview|Overview]] | [[00-Foundations/01-cloud-computing/README|Cloud Computing Index]]  
> **Status**: Complete & Verified

---

## Module Overview

Data is the ultimate asset of any enterprise, and storage is where data lives. Understanding storage hardware mechanics, filesystem indexing, redundancy algorithms, and cloud lifecycle economics enables engineers to architect durable, low-latency, and cost-optimized data systems.

This module delivers a comprehensive engineering reference for storage systems—spanning physical NAND flash physics and NVMe protocols up through POSIX filesystems, Reed-Solomon erasure coding, REST-accessible object stores, and multi-region disaster recovery.

```
00-Foundations/07-storage-fundamentals/
├── README.md                                          (This Index)
├── 01_Storage_Types_Block_File_Object.md              (The storage triumvirate: Block vs. File vs. Object)
├── 02_Hard_Drives_SSDs_and_NVMe.md                    (HDD mechanics, NAND flash physics, NVMe vs SATA AHCI)
├── 03_RAID_and_Storage_Redundancy.md                  (RAID 0-10, parity XOR, Erasure Coding, 11 9's durability)
├── 04_File_Systems_and_Protocols.md                   (POSIX, inodes, dentries, Ext4 vs XFS vs ZFS, NFS vs SMB)
├── 05_Object_Storage_Architecture.md                  (Flat namespaces, REST APIs, multipart uploads, strong consistency)
└── 06_Storage_Tiering_Lifecycle_and_Backup.md         (Hot/Cool/Cold tiering, incremental snapshots, RPO/RTO DR)
```

---

## Detailed Curriculum Index

1. [[01_Storage_Types_Block_File_Object|01 · Storage Types – Block, File, and Object Storage]]
   - The fundamental storage triumvirate: Block (EBS/Disk), File (NFS/SMB), and Object (S3/Blob).
   - Addressing mechanisms: LBA sectors, hierarchical paths, and flat key-value pairs.
   - Concurrency models: Single-attach (RWO) vs. multi-attach shared NAS (RWX).
   - Master technical comparison matrix across latency, throughput, and scale.
2. [[02_Hard_Drives_SSDs_and_NVMe|02 · Storage Media Internals – HDDs, SSDs, and NVMe]]
   - Mechanical HDD physics: Platters, actuators, rotational delay, and seek latency.
   - NAND Flash Cell architectures: SLC, MLC, TLC, and QLC endurance trade-offs.
   - Flash Erase Block asymmetry, Garbage Collection, and Write Amplification Factor (WAF).
   - Legacy SATA AHCI vs. modern parallel NVMe PCIe protocol.
   - Storage performance metrics: Mathematical relationship connecting IOPS, block size, and throughput.
3. [[03_RAID_and_Storage_Redundancy|03 · RAID and Storage Redundancy – Erasure Coding and Parity]]
   - Statistical disk failure models: Annualized Failure Rate (AFR) and Unrecoverable Read Errors (URE).
   - Classical RAID architectures: Striping (RAID 0), Mirroring (RAID 1), Parity (RAID 5/6), and RAID 10.
   - Bitwise XOR parity mathematics and the modern multi-terabyte rebuild catastrophe.
   - Reed-Solomon ($K + M$) Erasure Coding in cloud object stores (Ceph, S3).
   - The statistical mathematics behind "Eleven Nines" (99.999999999%) annual durability.
4. [[04_File_Systems_and_Protocols|04 · File Systems and Protocols – Ext4, XFS, ZFS, NFS, and SMB]]
   - The Virtual File System (VFS) abstraction layer in the Linux kernel.
   - Unix filesystem anatomy: Superblocks, Inodes, Directory Entries (`dentries`), and Data Blocks.
   - Hard links vs. Symbolic Links (Symlinks).
   - Filesystem Journaling (Write-Ahead Logging) and crash consistency.
   - Ext4 vs. XFS vs. ZFS comparison and network protocols (NFSv4 vs. SMB 3.1.1).
5. [[05_Object_Storage_Architecture|05 · Object Storage Architecture – Flat Namespaces, Buckets, and REST APIs]]
   - Object storage philosophy: Replacing directories with global flat key-value namespaces.
   - Core REST API primitives: `PUT`, `GET`, `HEAD`, `DELETE`, and Byte-Range reads.
   - Multipart Upload mechanics for reliable parallel ingestion of large payloads.
   - The Data Consistency revolution: Shift from eventual consistency to strong read-after-write consistency.
   - Regulatory compliance: WORM immutability and S3 Object Lock (Governance vs. Compliance modes).
6. [[06_Storage_Tiering_Lifecycle_and_Backup|06 · Storage Tiering, Lifecycle Management, and Disaster Recovery]]
   - The economic storage tiering spectrum: Hot (Standard), Cool (Infrequent), Cold (Archive), and Deep Archive.
   - Automated Lifecycle Management policies and minimum retention billing guardrails.
   - Incremental block snapshot chaining and Copy-on-Write (CoW) point-in-time recovery.
   - Business continuity metrics: Recovery Point Objective (RPO) and Recovery Time Objective (RTO).
   - Master comparison of cloud disaster recovery strategies: Backup & Restore, Pilot Light, Warm Standby, and Multi-Region Active-Active.

---

## Next Steps

> **Master Overview**: Return to [[Overview|Master Cloud Computing Overview]]  
> **Foundational Index**: Review all foundation modules in [[00-Foundations/01-cloud-computing/README|Module 01: Cloud Computing]]
