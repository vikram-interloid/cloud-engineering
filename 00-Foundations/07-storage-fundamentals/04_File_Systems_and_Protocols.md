# 04 · File Systems and Protocols – Ext4, XFS, ZFS, NFS, and SMB

> **Status**: Comprehensive Reference  
> **Parent**: [[Overview|Overview]] | [[00-Foundations/07-storage-fundamentals/README|Storage Index]]  
> **Topics**: Filesystem Architecture, Inodes & Superblocks, Dentries, Hard Links vs. Symlinks, Journaling Mechanics, Ext4 vs. XFS vs. ZFS, Network File Protocols (NFSv4 vs. SMB 3)

---

## 1. What Is a File System?

Raw block storage (an NVMe SSD or an EBS volume) is simply an unorganized array of billions of numbered 4 KB sectors. A **File System** is the software subsystem within the operating system kernel that organizes, indexes, protects, and retrieves files across those blocks.

```mermaid
flowchart TD
    subgraph UserSpace["User Space Application"]
        App["Application: Reads file '/var/log/app.log' via libc open()"]
    end

    subgraph VFS["Virtual File System (VFS) Layer in Linux Kernel"]
        VFS_Core["VFS Abstract Interface (POSIX standards: open, read, write, stat)"]
    end

    subgraph ConcreteFS["Concrete Filesystem Driver"]
        Driver["Ext4 / XFS / ZFS Kernel Driver"]
        Inodes["Inode Table (Translates file to block list: Blocks 401, 402, 580)"]
    end

    subgraph BlockLayer["Block Device Layer"]
        Disk["Underlying Block Device (/dev/nvme0n1 / EBS Volume)"]
    end

    App ==> VFS ==> ConcreteFS ==> Inodes ==> BlockLayer

    style UserSpace fill:#0284c7,stroke:#0284c7,stroke-width:1.5px,fill-opacity:0.08
    style VFS fill:#8b5cf6,stroke:#8b5cf6,stroke-width:1.5px,fill-opacity:0.08
    style ConcreteFS fill:#10b981,stroke:#10b981,stroke-width:1.5px,fill-opacity:0.08
    style BlockLayer fill:#f59e0b,stroke:#f59e0b,stroke-width:1.5px,fill-opacity:0.08

    classDef nodeStyle fill:#0f172a,stroke:#38bdf8,stroke-width:1px,fill-opacity:0.15,color:inherit;
    class App,VFS_Core,Driver,Inodes,Disk nodeStyle;
    linkStyle default stroke:#64748b,stroke-width:2px;
```

---

## 2. Anatomy of a Linux Filesystem: Inodes, Dentries, and Data Blocks

A common misunderstanding is that a file's name and its data are stored together. In Unix/Linux filesystems, they are strictly separated into three distinct structures:

```mermaid
flowchart LR
    subgraph Dentry["Directory Entry (dentry)"]
        direction TB
        D1["File Name: <b>report.pdf</b>"]
        D2["Points to: <b>Inode #1048576</b>"]
        D1 --- D2
    end

    subgraph InodeStruct["Inode #1048576 (Metadata Object)"]
        direction TB
        I1["File Size: 8,192 Bytes"]
        I2["Permissions: -rw-r--r-- (0644)"]
        I3["Owner: UID 1000 (alice) / GID 1000"]
        I4["Timestamps: atime, mtime, ctime"]
        I5["Block Pointers: [Block 5200, Block 5201]"]
        I6["<b>NOTE: Inode DOES NOT contain file name!</b>"]
        I1 --- I2 --- I3 --- I4 --- I5 --- I6
    end

    subgraph StorageBlocks["Physical / Virtual Storage Blocks"]
        direction TB
        B1["Disk Block 5200 (First 4KB of PDF binary data)"]
        B2["Disk Block 5201 (Second 4KB of PDF binary data)"]
        B1 --- B2
    end

    Dentry ==> InodeStruct ==> StorageBlocks

    style Dentry fill:#0284c7,stroke:#0284c7,stroke-width:1.5px,fill-opacity:0.08
    style InodeStruct fill:#8b5cf6,stroke:#8b5cf6,stroke-width:1.5px,fill-opacity:0.08
    style StorageBlocks fill:#10b981,stroke:#10b981,stroke-width:1.5px,fill-opacity:0.08

    classDef nodeStyle fill:#0f172a,stroke:#38bdf8,stroke-width:1px,fill-opacity:0.15,color:inherit;
    class D1,D2,I1,I2,I3,I4,I5,I6,B1,B2 nodeStyle;
    linkStyle default stroke:#64748b,stroke-width:2px;
```

### Hard Links vs. Symbolic Links (Symlinks)
- **Hard Link (`ln source target`)**: Creates a second Directory Entry (`dentry`) pointing to the **exact same inode number**. The link count in the inode increments. If you delete the original filename, the file data remains intact until all hard links are deleted and the link count hits zero.
- **Symbolic Link (`ln -s source target`)**: Creates an entirely new inode with its own data block that merely contains the string text path to the target file. If the target is deleted, the symlink breaks ("dangling link").

---

## 3. Filesystem Journaling: Preventing Corruption

When saving a file, an operating system must perform three distinct writes:
1. Write the new data to data blocks.
2. Update the inode with the new size and block pointers.
3. Update the directory entry with timestamp modifications.

If power fails between steps 1 and 2, the filesystem enters an inconsistent, corrupt state. **Journaling** solves this using a dedicated **Write-Ahead Log (WAL)**:

```mermaid
sequenceDiagram
    autonumber
    participant App as User Application
    participant FS as Journaling Filesystem (Ext4 / XFS)
    participant Log as Circular On-Disk Journal
    participant Data as Primary Filesystem Blocks

    App->>FS: write() new file data
    FS->>Log: Step 1: Writes transaction metadata to Journal
    FS->>Log: Step 2: Commits transaction in Journal (atomic commit)
    FS->>Data: Step 3: Writes actual data blocks & updates inode (Checkpointing)
    FS-->>Log: Step 4: Releases transaction from Journal

    Note over FS,Log: If power fails during Step 3, the kernel replays the Journal in seconds on reboot!
```

---

## 4. Master Comparison: Modern Linux Filesystems

| Feature | Ext4 (Fourth Extended FS) | XFS (Extended Filesystem) | ZFS (Zettabyte Filesystem) |
| :--- | :--- | :--- | :--- |
| **Architectural Model**| Classical Journaled Block FS | High-Performance 64-bit Journaled | Integrated Volume Manager + CoW |
| **Default Inode Allocation**| Fixed at filesystem creation (`mkfs`)| **Dynamic Inodes** (Allocated on demand)| Dynamic (Unlimited) |
| **Large File / Thread Scaling**| Good | **Superior** (Allocation Groups allow parallel multi-threaded writes)| **Superior** (Copy-on-Write) |
| **Max Single File Size**| 16 Terabytes | **8 Exabytes** | **16 Exabytes** |
| **Data Integrity Checking**| Metadata checksums only | Metadata checksums only | **End-to-End Cryptographic Block Checksums** (Fights bitrot) |
| **Snapshots** | No native snapshots | No native snapshots | **Instantaneous CoW Snapshots** |
| **Standard Cloud Role**| Ubuntu root OS volumes | RHEL/CentOS root volumes, high-performance databases | High-end storage appliances, multi-disk NAS arrays |

---

## 5. Network File Sharing Protocols: NFS vs. SMB

To share a filesystem across multiple virtual machines in the cloud, operating systems use network file protocols:

```mermaid
flowchart LR
    subgraph NFS_Stack["Linux Ecosystem · NFSv4 (RFC 7530)"]
        direction TB
        N_T["<b>NFSv4 Protocol</b>"]
        N_1["Standard for Linux / UNIX cloud fleets (AWS EFS)"]
        N_2["Runs over <b>TCP Port 2049</b> (Eliminates legacy portmapper)"]
        N_3["Stateful connection with integrated file locking and compound RPC requests"]
        N_T --- N_1 --- N_2 --- N_3
    end

    subgraph SMB_Stack["Windows Ecosystem · SMB 3.1.1"]
        direction TB
        S_T["<b>SMB / CIFS Protocol</b>"]
        S_1["Standard for Windows Server / Azure Files"]
        S_2["Runs over <b>TCP Port 445</b>"]
        S_3["Deep integration with Microsoft Active Directory and NTFS ACL permissions"]
        S_T --- S_1 --- S_2 --- S_3
    end

    style NFS_Stack fill:#0284c7,stroke:#0284c7,stroke-width:1.5px,fill-opacity:0.08
    style SMB_Stack fill:#10b981,stroke:#10b981,stroke-width:1.5px,fill-opacity:0.08

    classDef nodeStyle fill:#0f172a,stroke:#38bdf8,stroke-width:1px,fill-opacity:0.15,color:inherit;
    class N_T,N_1,N_2,N_3,S_T,S_1,S_2,S_3 nodeStyle;
    linkStyle default stroke:#64748b,stroke-width:2px;
```

---

## 6. Key Takeaways

| Filesystem Dimension | Core Engineering Principle |
| :--- | :--- |
| **Inode Separation** | Inodes store all file metadata (size, permissions, block pointers) but **never store the file name**, which lives in directory entries (`dentries`). |
| **No Inodes Left Error** | A disk can run out of space even with 50% free disk capacity if millions of tiny files exhaust the fixed inode table (`df -i`). |
| **Journaling Integrity** | Write-Ahead Logging protects against catastrophic filesystem corruption during abrupt power loss. |
| **XFS vs. Ext4 in Cloud** | XFS scales superiorly for high-throughput multi-threaded database workloads due to independent allocation groups. |

---

## 7. Knowledge Check

1. What is the fundamental difference between a block device and a file system?
2. What is an Inode, and what specific attributes does it record?
3. Why is the filename of a file stored in a Directory Entry (`dentry`) rather than inside the inode itself?
4. What happens when an operating system runs out of available inodes even though several gigabytes of disk space remain free?
5. Explain the mechanical difference between a Hard Link and a Symbolic Link (Symlink).
6. What is the operational purpose of filesystem Journaling, and how does it prevent lengthy `fsck` recovery scans on reboot?
7. In what workload scenario does the XFS filesystem significantly outperform Ext4?
8. How does ZFS achieve end-to-end data integrity against silent data corruption (bitrot)?
9. Which transport protocol and port does NFSv4 utilize for cloud shared file access?
10. In what enterprise scenarios would an architect choose SMB 3.1.1 (Azure Files) over NFSv4 (AWS EFS)?

---

## 8. Next Chapter

Continue your storage fundamentals learning journey:

> ➡️ **Next Module**: [[05_Object_Storage_Architecture|05 · Object Storage Architecture – Flat Namespaces, Buckets, and REST APIs]]  
> *Exploring flat key-value object namespaces, HTTP REST operations (GET/PUT/DELETE), multipart chunked uploads, and the transition from eventual consistency to strong read-after-write consistency.*

---

### Authoritative References

- [IETF RFC 7530: Network File System (NFS) Version 4 Protocol](https://datatracker.ietf.org/doc/html/rfc7530)
- [Linux Kernel Documentation: Filesystems in the Linux Kernel](https://docs.kernel.org/filesystems/index.html)
- [McKusick and Neville-Neil: The Design and Implementation of the FreeBSD Operating System (ZFS Chapters)](https://www.pearson.com/)
