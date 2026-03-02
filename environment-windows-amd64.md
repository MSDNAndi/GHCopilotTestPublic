# Coding Agent Environment — Windows AMD64

This document describes the hardware and runtime constraints of the **current** GitHub Copilot Coding Agent environment.

> **Note:** The previous Windows/ARM64 environment is documented in [`environment-windows-arm.md`](./environment-windows-arm.md).  
> **Software environment** (languages, compilers, tools): See [`environment-windows-amd64-software.md`](./environment-windows-amd64-software.md).

---

## Platform

| Property        | Value                                                |
|-----------------|------------------------------------------------------|
| OS              | Microsoft Windows Server 2025 Datacenter             |
| OS Version      | 10.0.26100 (Build 26100)                             |
| Edition         | Datacenter                                           |
| Runner label    | `windows-2025`                                       |
| Runner arch     | `X64`                                                |
| Image version   | `20260225.38.2`                                      |
| Virtualisation  | Microsoft Hyper-V (fully virtualised)                |
| System type     | Virtual Machine (x64-based PC)                       |

---

## CPU

| Property             | Value                                              |
|----------------------|----------------------------------------------------|
| Processor            | AMD EPYC 7763 64-Core Processor                    |
| Architecture         | AMD64 (x86_64)                                     |
| Family/Model         | AMD64 Family 25 Model 1 Stepping 1, AuthenticAMD   |
| Manufacturer         | AuthenticAMD                                       |
| Physical sockets     | 1                                                  |
| Physical cores       | 2                                                  |
| Logical CPUs         | 4 (2 physical × 2 SMT/Hyper-Threading threads)     |
| Max clock speed      | 2,445 MHz                                          |
| L2 cache             | 1 MB                                               |
| L3 cache             | 32 MB                                              |

AMD EPYC 7763 is a server-grade Zen 3 processor used in Azure infrastructure. The VM exposes 4 logical CPUs — 2 physical cores with Hyper-Threading enabled.

---

## Memory (RAM)

| Property                     | Value        |
|------------------------------|--------------|
| Total physical RAM           | 16.0 GB      |
| Page file location           | `D:\pagefile.sys` |
| Page file size               | 2.94 GB      |
| Total addressable (RAM + PF) | 18.87 GB     |

---

## Storage

### Physical Disks

Two physical disks are visible to the VM:

| Disk | Model                        | Interface | Style | Size     | Partitions | Role              |
|------|------------------------------|-----------|-------|----------|------------|-------------------|
| 0    | Msft Virtual Disk            | SAS/SCSI  | GPT   | 150.0 GB | 4          | OS / boot disk    |
| 1    | Msft Virtual Disk            | SAS/SCSI  | MBR   | 150.0 GB | 1          | Temporary Storage |

> **Disk 0 (GPT)** hosts the Windows OS on `C:`.  
> **Disk 1 (MBR)** is formatted as NTFS and mounted as `D:` labelled **"Temporary Storage"**. It is pre-formatted and ready for use. The page file (`D:\pagefile.sys`) resides here.

### Disk 0 Partition Layout (GPT)

| Partition | Type          | Size    | Offset  | Filesystem | Drive | Label    | Notes           |
|-----------|---------------|---------|---------|------------|-------|----------|-----------------|
| 1         | Reserved      | 16 MB   | 1 MB    | —          | —     | —        | GPT reserved    |
| 2         | Recovery      | 450 MB  | 17 MB   | NTFS       | —     | Recovery | Hidden, WinRE   |
| 3         | System (EFI)  | 99 MB   | 467 MB  | FAT32      | —     | —        | EFI boot files  |
| 4         | Primary (OS)  | 149.4 GB| 566 MB  | NTFS       | `C:`  | Windows  | Boot / OS volume |

### C: Volume (NTFS) — OS Disk

| Property                      | Value                          |
|-------------------------------|--------------------------------|
| Volume label                  | Windows                        |
| Filesystem                    | NTFS 3.1 (LFS 2.0)             |
| Volume serial number          | `FA688D28688CE4AB`              |
| Cluster size                  | 4,096 bytes (4 KB)             |
| Bytes per sector              | 512                            |
| Bytes per physical sector     | 4,096                          |
| Total clusters                | 39,176,699                     |
| Total capacity                | 149.45 GB                      |
| MFT valid data length         | ~1.21 GB                       |
| MFT zone size                 | ~112 MB                        |
| **Free at session start**     | **~33 GB** (varies by session) |

### D: Volume (NTFS) — Temporary Storage

| Property                      | Value                            |
|-------------------------------|----------------------------------|
| Volume label                  | Temporary Storage                |
| Filesystem                    | NTFS                             |
| Cluster size                  | 4,096 bytes (4 KB)               |
| Total capacity                | 150.0 GB                         |
| **Free at session start**     | **~146–148 GB** (varies by session) |

> `D:` is a dedicated scratch disk with nearly all space available. The page file (`D:\pagefile.sys`, ~2.94 GB) is pre-allocated here.

---

## Timeouts

| Timeout                           | Value      | Source                              |
|-----------------------------------|------------|-------------------------------------|
| Coding Agent maximum session time | 59 minutes | `COPILOT_AGENT_TIMEOUT_MIN=59` env var |

The `COPILOT_AGENT_TIMEOUT_MIN` environment variable is set to **59**, meaning the Copilot Coding Agent will be terminated after a maximum of **59 minutes** per session.

---

## Empirical Tests

The sections below document results of stress tests run to characterise limits on CPU parallelism, RAM, and disk capacity.

---

### CPU Test — Parallelism & Threading

**Method:** Python 3.12 (AMD64 x64) benchmark using `concurrent.futures.ProcessPoolExecutor` (process-based) and `threading.Thread` (thread-based). Each worker executes 10 million floating-point `sqrt` iterations. Serial baseline measured first.

**Environment:** Python 3.12.10 (AMD64 x64), `os.cpu_count() = 4`.

#### Process-based scaling (true parallelism — no GIL)

| Workers (N) | Wall time | Throughput speedup | Efficiency |
|-------------|-----------|-------------------|------------|
| 1           | 0.875 s   | 0.90×             | 90%        |
| 2           | 0.973 s   | 1.61×             | 81%        |
| 3           | 1.398 s   | 1.69×             | 56%        |
| 4           | 1.696 s   | 1.85×             | 46%        |

> Serial baseline: **0.786 s** (single-threaded, no pool overhead).
>
> The 1-worker pool is slightly slower than serial due to `ProcessPoolExecutor` process-spawn overhead. N=2 achieves the best efficiency (81%), matching the 2 physical cores. N=3 and N=4 show diminishing returns because the extra logical CPUs are Hyper-Threading siblings — the additional threads share execution resources on the same physical cores.

#### Thread-based scaling (GIL-limited for CPU-bound Python code)

| Threads (N) | Wall time | Relative to serial |
|-------------|-----------|-------------------|
| 1           | 0.781 s   | 1.0× (baseline)   |
| 2           | 1.825 s   | 2.3× **slower**   |
| 3           | 2.491 s   | 3.2× **slower**   |
| 4           | 3.323 s   | 4.2× **slower**   |

Python threads scale **linearly worse** with thread count for CPU-bound work due to the **Global Interpreter Lock (GIL)**. The GIL allows only one thread to execute Python bytecode at a time, so N threads each contend for the lock and total wall time grows as N × serial_time.

**Conclusion:**
- **2 physical cores** with Hyper-Threading → **4 logical CPUs** total.
- Best process-based speedup: **1.85× with 4 processes** (but only 1.61× at N=2 with 81% efficiency — the HT topology means N=2 is more efficient).
- **Python threads do not provide CPU parallelism** for CPU-bound workloads due to the GIL.
- For I/O-bound workloads, threads are fully effective (GIL is released during I/O).

---

### RAM Test — Allocation to Exhaustion

**Method:** Python `bytearray` allocations with every 4 KB page touched to force physical mapping. `GlobalMemoryStatusEx` (Win32 API) used to report available RAM and page-file usage after each allocation. Allocation was freed between steps (`del buf; gc.collect()`).

**Starting state:** 16.0 GB physical RAM, 2.94 GB page file, ~12.86 GB available at test start.

| Allocated | Outcome | Avail RAM after | Avail PF after | Notes |
|-----------|---------|-----------------|----------------|-------|
| 1 GB      | ✅ OK   | 11.61 GB        | 14.67 GB       |       |
| 4 GB      | ✅ OK   | 8.60 GB         | 11.67 GB       |       |
| 8 GB      | ✅ OK   | 4.73 GB         | 7.66 GB        |       |
| 12 GB     | ✅ OK   | 1.42 GB         | 3.66 GB        |       |
| 13 GB     | ✅ OK   | 1.03 GB         | 2.09 GB        | ⚠ RAM nearly full — spilling to PF |
| 14 GB     | ✅ OK   | 1.04 GB         | 1.02 GB        | Majority in page file |
| 15 GB     | ✅ OK   | 1.47 GB         | 1.47 GB        | Page file heavily used |
| 16 GB     | ✅ OK   | 1.29 GB         | 1.27 GB        | Beyond physical RAM ceiling |
| 17 GB     | ✅ OK   | 1.01 GB         | 0.95 GB        | Well beyond RAM+PF advertised limit |
| 18 GB     | ✅ OK   | 2.18 GB         | 1.98 GB        | OS reclaims cache aggressively |
| 19 GB     | ✅ OK   | 5.85 GB         | 5.61 GB        | No MemoryError — test limit reached |

**Last successful allocation: 19.00 GB** (no `MemoryError` was triggered up to 19 GB).

**Observations:**
- Allocations beyond ~13 GB begin to use the page file heavily.
- The OS reports 18.87 GB total virtual memory (RAM + PF), yet 19 GB was allocatable — the OS compresses pages and reclaims cache to accommodate allocations beyond the nominal ceiling.
- No artificial per-process memory cap was detected.

**Conclusion:** The full ~16 GB physical RAM plus ~3 GB page file is accessible. **There is no hidden per-process memory cap below the physical hardware ceiling.**

---

### Disk Test — Full Fill Capacity & Write Speed

**Method:** Write 1 GB files sequentially to `D:\disk_fill_test\`, `fsync` after each file. Stop when ≤ 2 GB remains (safety margin). Chunk size reduced to 256 MB when free space falls below 5 GB.

**Drive used:** `D:` (Temporary Storage) — chosen because it has ~148 GB free vs only ~33 GB on `C:`.

**Starting state:** 148.41 GB free on `D:`.

#### Fill progress (every 10 GB milestone)

| Written total | Free after   | Write speed | Notes                       |
|---------------|--------------|-------------|-----------------------------|
| 10.7 GB       | 137.67 GB    | ~399 MB/s   |                             |
| 21.5 GB       | 126.93 GB    | ~397 MB/s   |                             |
| 32.2 GB       | 116.19 GB    | ~398 MB/s   |                             |
| 42.9 GB       | 105.46 GB    | ~397 MB/s   |                             |
| 53.7 GB       | 94.72 GB     | ~403 MB/s   |                             |
| 64.4 GB       | 83.98 GB     | ~398 MB/s   |                             |
| 75.2 GB       | 73.24 GB     | ~405 MB/s   |                             |
| 85.9 GB       | 62.51 GB     | ~392 MB/s   |                             |
| 96.6 GB       | 51.77 GB     | ~406 MB/s   |                             |
| 107.4 GB      | 41.03 GB     | ~403 MB/s   |                             |
| 118.1 GB      | 30.30 GB     | ~406 MB/s   |                             |
| 128.8 GB      | 19.56 GB     | ~405 MB/s   |                             |
| 139.6 GB      | 8.82 GB      | ~405 MB/s   |                             |
| 146.3 GB      | **2.11 GB**  | **~408 MB/s** | ← **safety margin hit — stopped** |

> Write speed was **consistently 390–420 MB/s** from the very first gigabyte all the way to 146 GB — no degradation at any fill level.

#### Final state

| Metric                        | Value         |
|-------------------------------|---------------|
| Initial free space (`D:`)     | 148.41 GB     |
| **Total successfully written**| **146.30 GB** |
| Free at stop (safety margin)  | 2.11 GB       |
| Free after cleanup            | 148.41 GB     |
| Write speed range             | ~390–420 MB/s |
| Errors / quota events         | None          |

#### Conclusions

- **146.30 GB** was written without any `OSError`, quota error, or throttling.
- Write speed was **completely stable** throughout — no caching artifact or NVMe saturation was observed.
- The full ~148 GB of free disk space on `D:` is entirely usable; the only limit is raw volume capacity.
- Cleanup (`shutil.rmtree`) restored the full free space instantly.

---

## Summary — Key Differences vs Previous Windows/ARM64 Environment

| Dimension            | Previous (Windows/ARM64)                  | Current (Windows/AMD64)                   |
|----------------------|-------------------------------------------|-------------------------------------------|
| OS                   | Windows 10/11 Enterprise ARM64            | Windows Server 2025 Datacenter AMD64      |
| Runner label         | `windows-11-arm`                          | `windows-2025`                            |
| CPU                  | Microsoft Cobalt 100 (ARM64)              | AMD EPYC 7763 (AMD64 / x86_64)            |
| Logical CPUs         | 4 (4 physical, no SMT)                    | 4 (2 physical × 2 HT threads)             |
| CPU topology         | 4 independent physical cores              | 2 cores × 2 HT threads                    |
| Max clock speed      | 3,399 MHz                                 | 2,445 MHz                                 |
| L3 cache             | 128 MB                                    | 32 MB                                     |
| CPU parallelism      | Up to 2.61× at N=3 (87% eff)             | Up to 1.85× at N=4 (best eff at N=2: 81%) |
| Total RAM            | 15.99 GB + 2.88 GB page file              | 16.0 GB + 2.94 GB page file               |
| Max allocatable      | ≥19 GB (no MemoryError found)             | ≥19 GB (no MemoryError found)             |
| C: free at start     | ~119–123 GB                               | ~33 GB (**much less**)                    |
| D: drive             | N/A (no D:)                               | 150 GB NTFS "Temporary Storage" (~148 GB free) |
| Write speed (D:)     | —                                         | ~390–420 MB/s                             |
| Write speed (C: ARM) | ~120–155 MB/s                             | —                                         |
| Docker               | ❌ Not available                           | ✅ Docker Engine 29.1.5                   |
| Mercurial (`hg`)     | ✅ 6.3.1                                  | ❌ Not available                           |
| Session timeout      | 59 minutes                                | 59 minutes (unchanged)                    |
