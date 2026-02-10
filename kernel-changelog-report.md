# Linux Kernel Version Changelog Report

> Source: [kernelnewbies.org](https://kernelnewbies.org/LinuxVersions)
> Coverage: Linux 6.13 – 6.19 (January 2025 – Early 2026)
> Generated: 2026-02-10

---

## Release Timeline

| Version | Release Date    | Development Cycle |
|---------|-----------------|-------------------|
| 6.19    | ~Dec 2025       | ~63 days          |
| 6.18    | ~Nov 2025       | ~63 days          |
| 6.17    | Sept 28, 2025   | ~63 days          |
| 6.16    | July 27, 2025   | ~63 days          |
| 6.15    | May 15, 2025    | ~63 days          |
| 6.14    | Mar 24, 2025    | ~63 days          |
| 6.13    | Jan 19, 2025    | ~63 days          |

---

## Linux 6.19

### Headline Features
- **listns(2) system call** – New syscall for iterating through namespaces without scanning /proc; supports pagination, filtering, and permission checking
- **Live Update Orchestrator** – Kernel subsystem enabling live updates via kexec-based reboots while preserving system state; critical for cloud VM environments
- **PCIe Link Encryption** – Encrypts PCIe traffic between confidential VMs and devices, preventing host OS from intercepting DMA
- **Color Pipeline API** – Hardware-accelerated color transformations for HDR display support
- **SFrame Format** – Fast stack unwinding using a minimal format simpler than DWARF

### Filesystems
- **Btrfs**: Shutdown ioctl, suspendable scrub/device-replace, async checksum (+15% write perf), RAID56 block size expansion
- **Ext4**: Large block sizes beyond page size, online defrag optimization (~50% buffered IO write gains)
- **NFS/NFSD**: Direct I/O reads, multiple extents in LAYOUTGET, directory delegation enhancements

### Core Subsystems
- **io_uring**: Mixed-sized SQEs, zero-copy receive (zcrx) with shared interface queues, getsockname/getpeername
- **Scheduler**: MM CID rewrite, NEXT_BUDDY for EEVDF, sched_ext bypass scalability, proportional newidle balancing
- **Memory**: Device-private THP, dmabuf for iommufd, ECC without struct page, network throttling under memcg

### Rust Support
- Bounded integer types, debugfs binary blobs, extended module macro, PWM abstractions, basic I2C driver abstractions

---

## Linux 6.18

### Headline Features
- **Slub Sheaves** – Per-CPU cache mechanism eliminating synchronization overhead during memory allocation/freeing
- **dm-pcache** – Persistent memory device caching layer for traditional storage
- **Process Namespace Handles** – Encode/decode namespace file handles via name_to_handle_at()/open_by_handle_at()
- **Accurate ECN** – TCP provides granular congestion feedback signals per RTT
- **PSP Encryption Protocol** – Google's protocol with superior HW offload capabilities vs IPsec/TLS
- **BPF Program Signing** – Cryptographic signing enabling future unprivileged loading of vetted programs
- **Swap Performance** – New swap table infrastructure providing 5–20% throughput gains

### Filesystems
- **Btrfs**: Improved transaction commit via commit root checksum searches, compression prep for larger block sizes
- **XFS**: Online repair enhancements, V4 support disabled by default, online fsck enabled by default
- **Ext4**: Mounted superblock updates via ioctl (no block device writes for tune2fs)
- **FUSE**: COPY_FILE_RANGE_64, prune notifications, synchronous FUSE_INIT
- **OverlayFS**: Casefold layer support for case-insensitive operations

### Networking
- UDP stack optimizations with 50% additional performance gains (NUMA-aware locking, contention reduction)
- NFS server prototype disabling I/O caching for better scalability

### Memory Management
- Huge zero folio support, expanded anonymous page collapsing, per-VMA locking for /proc/pid/maps
- memdesc_flags_t introduction for future struct page/slab/folio separation

### Block Layer
- md-llbitmap: Lockless bitmap for RAID metadata tracking
- rnull: Configfs and remote completion support

---

## Linux 6.17

### Headline Features
- **CPU Bug Mitigation Selection** – Easier mitigation selection based on chosen attack vectors, auto-updated as new bugs are discovered
- **file_getattr()/file_setattr() syscalls** – Extended file attribute manipulation using openat(2) semantics
- **Priority Inheritance (Proxy Execution)** – Lower-priority tasks inherit scheduling context from higher-priority waiters
- **Unconditional SMP Scheduler** – Eliminated uniprocessor-specific scheduler paths
- **Per-Node Memory Reclaim** – NUMA per-node proactive reclaim controls beyond memcg
- **FALLOC_FL_WRITE_ZEROES** – Leverages SSD capabilities for efficient zero pre-allocation
- **Runtime Verification with LTL** – Linear temporal logic monitors for validating real-time application behavior

### Filesystems
- **Btrfs**: Compressed readahead improvements, free space tree optimization (~20% perf gain)
- **Ext4**: Block allocation scalability, uncached buffered I/O, FALLOC_FL_WRITE_ZEROES
- **NFS**: Write delegation improvements, async COPY re-enabled, client btime support
- **F2FS**: Mount API conversion, enhanced recovery stats, GC tuning options
- **FUSE**: iomap integration for buffered writes
- **EROFS**: Metadata compression, directory readahead

### Memory Management
- mprotect() speedups (>3x improvement) for large folios
- mremap() optimization (37% execution time reduction for large folios)
- Per-vma lock for /proc/pid/maps reads
- Multi-VMA mremap() for huge folios, mTHP swap improvements

### Tracing & BPF
- Tracing link cookies, bpf_dynptr_memset(), bpf_token_info, bpf_cgroup_read_xattr
- Memory accounting for BPF programs, uid filtering via BPF filters
- Graph tracer enhancements (args, retval, retaddr options)
- Dynamic ftrace unconditionally enabled

### Rust Support
- Bug/warn abstractions, ACPI match tables, bits/genmask macros, device property reads, platform I/O, hrtimer Instant/Delta types

---

## Linux 6.16

### Headline Features
- **XFS Large Atomic Writes** – Multiple filesystem blocks written atomically
- **Intel TDX Support** – Initial Trusted Domain Extensions for confidential guest VMs
- **Device Memory TCP TX** – Zero-copy TCP transmission from DMABUF/GPU memory
- **USB Audio Offload** – Power-efficient embedded audio streaming
- **Automatic Weighted Interleave** – NUMA auto-tuning memory allocation policy
- **Kexec Handover (KHO)** – Mechanism for preserving state across kexec

### Filesystems
- **Ext4**: Fast commit enhancements, multi-fsblock atomic writes for bigalloc, large folio support (+37% sequential I/O)
- **Bcachefs**: Single device mode, snapshot deletion improvements
- **NFS**: Local I/O support, fallocate improvements, FATTR4_CLONE_BLKSIZE
- **FUSE**: Large folio support, enhanced cache invalidation

### Virtualization
- TDX initialization with vCPU and VM creation
- Virtio RTC module, enhanced SEV/SNP, Hyper-V VTL Boot

### Security
- SELinux directory access cache, wildcard genfscon matching
- Kexec event measurement between load and execute

### Rust Support
- clk, cpumask, cpufreq, opp, xarray, mmap abstractions
- DRM Rust abstractions and Nova GPU driver support

### Architecture
- Intel APX (Advanced Performance Extensions) with 32 general-purpose registers

---

## Linux 6.15

### Headline Features
- **VFS Mount Namespace Expansion** – open_tree_attr() syscall for idmapped mounts from existing idmapped mounts; detached mount assembly for private rootfs
- **Mount Notifications via fanotify** – Listen for mount topology changes without /proc/pid/mountinfo
- **AMD INVLPGB TLB Invalidation** – Broadcast TLB invalidation without IPIs on Zen 3+
- **io_uring Zero-Copy Receive** – Fast bulk receive directly into application memory
- **Perf Latency Profiling** – Wall-time impact display rather than CPU-time
- **fwctl Subsystem** – Standardized firmware interface for debugging, config, and provisioning

### Filesystems
- **Bcachefs**: Scrubbing, block size > page size, case-folding
- **Btrfs**: Send path optimization (~30% improvement), zstd negative compression levels
- **XFS**: Large atomic writes with COW, zone GC threshold tuning, 30+ zoned device commits
- **Ext4**: Linear dentry search optimization, error handling improvements
- **F2FS**: Mount API conversion begun, nat_bits as mount option

### Memory Management
- Per-vma locking reimplemented with refcounting
- mseal system mapping support, guard regions for file-backed/shmem mappings
- defrag_mode sysctl for huge page maintenance
- Batched unmap lazyfree for large folios, per-CMA locking

### Scheduler Extensions (sched_ext)
- Core event counters, per-NUMA idle cpumask splits, SCX_OPS_ALLOW_QUEUED_WAKEUP

### Rust Support
- Pin-init as standalone crate, hrtimer APIs, mm_struct/vm_area_struct/mmap bindings

---

## Linux 6.14

### Headline Features
- **NT Synchronization Primitives** – Windows NT sync primitives on Linux for Wine/game emulation performance
- **Btrfs RAID1 Read Balancing** – Rotation, latency, and devid-based balancing methods
- **Uncached Buffered I/O** – Drop pages from cache after read/write on fast storage
- **fsnotify Pre-Access Events** – FS_PRE_ACCESS for on-demand content filling from slow storage
- **dmem cgroup for GPU** – GPU and driver-allocated memory tracked per cgroup
- **FUSE io_uring** – Reduced context switches for FUSE filesystem operations
- **AMD XDNA NPU Driver** – Machine learning execution (CNNs, LLMs) on AMD NPUs

### Filesystems
- **XFS**: Reflink and reverse-mapping for realtime devices
- Lockless mount namespace lookups, symlink length caching (+1.5% on ext4)
- pidfs file handle support and bind-mount capability

### Networking
- TCP time-wait reuse delay configurability
- SO_PRIORITY cmsg support, IGMP/MLD join/leave notifications
- VXLAN reserved bits customization, unified PHY statistics

### Security
- Script execution control with AT_EXECVE_CHECK
- Module signing with SHA512 (replacing SHA1)
- SELinux xperms in conditional policies

### Memory Management
- mglru performance optimizations, large folio support for tmpfs
- Page table accounting at all levels
- DAMON debugfs removal, filter extensions

---

## Linux 6.13

### Headline Features
- **Lazy Preemption Mode** – Balanced scheduling between voluntary and full preemption
- **Atomic Write Support** – XFS, Ext4 Direct I/O, and MD RAID atomic writes
- **Multi-Grain Timestamps** – Fine-grained file timestamps activated only when queried
- **NAPI Suspension** – Packet delivery alternating busy-poll/interrupt based on activity
- **Lightweight Guard Pages** – MADV_GUARD_INSTALL for memory-efficient protection
- **ARM64 CCA** – Confidential Compute Architecture for protected VMs
- **Guarded Control Stack** – Hardware-protected return addresses against ROP attacks

### Filesystems
- **Btrfs**: Delayed head references converted to xarray, reduced lock contention
- **XFS**: Allocation groups converted to xarray, metadata inode directory trees, realtime sharding
- **Tmpfs**: Case-insensitive support, 20% read performance improvement

### io_uring
- Ring resizing, fixed wait regions, hybrid IO polling, partial buffer cloning, static NAPI

### Networking
- TX hardware shaping API, per-NAPI configuration via netlink
- UDP four-tuple hash for connected sockets

### Memory Management
- Page allocation tag compression (overhead reduced from 41% to 5.5%)
- Large folio zswap with XArray optimization
- SLUB per-object memory policy support

### Rust Support
- PidNamespace, file operations for Binder, tracepoints, static branches, seqfile, PCI/platform device drivers

---

## Cross-Version Trend Analysis

### 1. Rust Language Integration
Rust support has expanded steadily across every release:
- **6.13**: PidNamespace, Binder file ops, tracepoints, PCI drivers
- **6.14**: Extended modversions allowing concurrent Rust + MODVERSIONS builds
- **6.15**: Pin-init standalone crate, hrtimer, mm_struct bindings
- **6.16**: clk, cpumask, cpufreq, DRM abstractions, Nova GPU driver
- **6.17**: ACPI, platform I/O, hrtimer Instant/Delta types
- **6.18**: debugfs, sg_table, iov_iter, maple tree, PCI, USB sample driver
- **6.19**: Bounded ints, debugfs blobs, PWM, I2C driver abstractions

### 2. io_uring Evolution
Each release adds new capabilities to the asynchronous I/O framework:
- **6.13**: Ring resizing, fixed wait regions, hybrid polling
- **6.14**: FUSE io_uring communication
- **6.15**: Zero-copy receive, epoll event reading
- **6.16**: (Continued improvements)
- **6.17**: Timestamp commands, vectorized sends
- **6.18**: Mixed-sized CQEs, zcrx, multishot uring_cmd
- **6.19**: Mixed-sized SQEs, zcrx with shared interface queues

### 3. Memory Management Advances
- Large folio support expanding from tmpfs (6.13) through filesystems and swap
- Per-VMA locking evolving from refcounting (6.15) to /proc reads (6.17, 6.18)
- DAMON improvements every release for data access monitoring
- Progressive struct page/slab/folio separation work

### 4. Filesystem Maturation
- **Atomic writes**: Started in XFS/Ext4 (6.13), expanded in XFS (6.16)
- **Btrfs**: Consistent improvements in RAID, compression, performance every release
- **Bcachefs**: Rapid maturation with scrubbing, single-device mode, snapshots
- **FUSE**: From io_uring support (6.14) to iomap integration (6.17) to large copies (6.18)

### 5. Confidential Computing
- **6.13**: ARM64 CCA for protected VMs
- **6.16**: Intel TDX initialization
- **6.18**: PSP encryption protocol
- **6.19**: PCIe link encryption for confidential VMs

### 6. Scheduling Evolution
- **6.13**: Lazy preemption mode
- **6.15**: sched_ext with core counters and NUMA awareness
- **6.17**: Proxy execution for priority inheritance, unconditional SMP
- **6.18**: Inlined migrate functions, deferred throttling
- **6.19**: EEVDF NEXT_BUDDY, MM CID rewrite

### 7. Network Performance
- UDP optimizations nearly every release (four-tuple hash, NUMA-aware locking, DDOS resilience)
- Zero-copy paths expanding: DMABUF TCP TX (6.16), io_uring zcrx (6.15+)
- Congestion control evolution: Accurate ECN (6.18)

---

## LTS (Long-Term Support) Versions in 6.x

| Version | LTS Status |
|---------|------------|
| 6.12    | LTS        |
| 6.6     | LTS        |
| 6.1     | LTS        |

---

## Summary Statistics (6.13–6.19)

- **7 releases** covering ~14 months of development
- **~63-day** average release cycle maintained consistently
- **Rust abstractions** added in every single release
- **io_uring** enhanced in every single release
- **Memory management** improvements in every single release
- Major new subsystems: Live Update Orchestrator, fwctl, dm-pcache, slub sheaves
- Major new syscalls: listns(2), open_tree_attr(), file_getattr()/file_setattr()
- Key security additions: TDX, CCA, PCIe encryption, BPF signing, guarded control stacks
