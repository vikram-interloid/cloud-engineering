# Module 06 · Compute Fundamentals for Cloud Engineers

> **Path**: `00-Foundations/06-compute-fundamentals/`  
> **Master Overview**: [[Overview|Overview]] | [[00-Foundations/01-cloud-computing/README|Cloud Computing Index]]  
> **Status**: Complete & Verified

---

## Module Overview

Compute is the execution engine of the cloud. From raw bare-metal Silicon and CPU cache hierarchies to Linux kernel namespaces, hardware-assisted hypervisors, and sub-5ms serverless microVM sandboxes, understanding compute mechanics is vital for writing performant, cost-effective distributed systems.

This module delivers a thorough engineering breakdown of computer architecture, operating system kernels, virtualization mechanics, container primitives, and scalable performance tuning.

```
00-Foundations/06-compute-fundamentals/
├── README.md                                          (This Index)
├── 01_Computer_Architecture_CPU_RAM_Bus.md            (von Neumann, CPU pipelines, L1-L3 caches, NUMA, PCIe)
├── 02_Operating_System_Kernels_and_Processes.md       (Ring 0 vs 3, syscalls, process memory, threads, CFS)
├── 03_Virtualization_and_Hypervisors.md               (Type-1 vs Type-2, Intel VT-x, SLAT/EPT, AWS Nitro)
├── 04_Containerization_and_Linux_Primitives.md        (Namespaces, cgroups v1/v2, OverlayFS, OCI runc)
├── 05_Serverless_Compute_and_MicroVMs.md              (Firecracker KVM, gVisor, Wasm, cold start, SnapStart)
└── 06_Scaling_and_Performance_Tuning.md               (Scale-up vs out, Amdahl's Law, USL, thread pool math)
```

---

## Detailed Curriculum Index

1. [[01_Computer_Architecture_CPU_RAM_Bus|01 · Computer Architecture – CPU, Memory Hierarchy, and System Bus]]
   - The classical von Neumann model (ALU, CU, Registers, Memory, I/O).
   - CPU instruction execution cycle: Fetch, Decode, Execute, Write-back.
   - The Memory Wall and the multi-tiered cache hierarchy (L1, L2, L3 vs. DDR4/DDR5 RAM).
   - Non-Uniform Memory Access (NUMA) in multi-socket cloud hypervisors.
   - PCIe Gen 4/5 bus architecture and throughput capabilities.
2. [[02_Operating_System_Kernels_and_Processes|02 · Operating System Kernels, Syscalls, and Processes]]
   - Hardware protection rings: Ring 3 User Space vs. Ring 0 Kernel Space.
   - System calls (`syscall`/`sysret`), interrupt vectors, and execution overhead.
   - Process memory segments (Text, Data, BSS, Heap, Stack).
   - Process vs. Thread architecture and context switching penalties.
   - Linux Completely Fair Scheduler (CFS), Red-Black trees, and `vruntime`.
3. [[03_Virtualization_and_Hypervisors|03 · Virtualization and Hypervisors – Type-1 vs. Type-2]]
   - The Popek-Goldberg virtualization theorem and x86 hardware trap limitations.
   - Intel VT-x / AMD-V: VMX Root vs. VMX Non-Root execution and VMCS.
   - Second-Level Address Translation (SLAT / Intel Extended Page Tables).
   - Type-1 bare-metal hypervisors (KVM, ESXi) vs. Type-2 hosted hypervisors.
   - Memory management (Ballooning, THP, KSM) and the AWS Nitro architecture.
4. [[04_Containerization_and_Linux_Primitives|04 · Containerization and Linux Primitives – Namespaces, cgroups, and OverlayFS]]
   - The architectural reality: "Containers are just isolated Linux processes."
   - Linux Namespaces (PID, NET, MNT, IPC, UTS, USER, CGROUP).
   - Control Groups (cgroups v1/v2): CPU CFS quota throttling and OOM Killer mechanics.
   - OverlayFS union mount mechanics: LowerDir, UpperDir, MergedDir, and Copy-on-Write.
   - The Open Container Initiative (OCI), containerd, and runc.
5. [[05_Serverless_Compute_and_MicroVMs|05 · Serverless Compute and MicroVMs – Firecracker, gVisor, and Wasm]]
   - The security isolation gap: Shared kernel containers vs. heavyweight VMs.
   - AWS Firecracker: Stripped hardware emulation, Rust memory safety, sub-5ms boot.
   - Alternative sandboxes: Google gVisor syscall virtualization and WebAssembly (Wasm).
   - Serverless cold start anatomy: Provisioning, runtime init, handler execution.
   - Cold start acceleration: AWS SnapStart memory and disk snapshot restoration.
6. [[06_Scaling_and_Performance_Tuning|06 · Scaling and Performance Tuning – Amdahl's Law and Concurrency]]
   - Vertical scaling (Scale-Up) vs. Horizontal scaling (Scale-Out).
   - Amdahl's Law and the non-parallelizable serial bottleneck ceiling.
   - Neil Gunther's Universal Scalability Law (USL) and retrograde cluster scaling.
   - Concurrency (structural management) vs. Parallelism (physical multi-core execution).
   - Goetz's thread pool sizing formula and Linux load average interpretation.

---

## Next Steps

> ➡️ **Proceed to Module 07**: [[00-Foundations/07-storage-fundamentals/README|07 · Storage Fundamentals]]  
> *Dive deep into the storage triumvirate (Block, File, Object), NVMe SSD flash architectures, RAID vs. Erasure Coding (Reed-Solomon), and cloud object storage consistency models.*
