# Coding Agent Environment — Windows ARM64

This document describes the hardware and runtime constraints of the **current** GitHub Copilot Coding Agent environment.

> **Note:** The previous Linux/AMD64 environment is documented in [`environment.md`](./environment.md).  
> **Software environment** (languages, compilers, tools): See [`environment-windows-arm-software.md`](./environment-windows-arm-software.md).

---

## Platform

| Property        | Value                                              |
|-----------------|----------------------------------------------------|
| OS              | Windows 10 Enterprise (Windows 11 release channel) |
| OS Version      | 10.0.26200.7462 (Windows 11 25H2 equivalent)       |
| Edition         | Enterprise                                         |
| Runner label    | `windows-11-arm`                                   |
| Virtualisation  | Microsoft Hyper-V (fully virtualised)              |
| System type     | Virtual Machine                                    |

---

## CPU

| Property             | Value                                        |
|----------------------|----------------------------------------------|
| Processor            | Microsoft Cobalt 100                         |
| Architecture         | ARM64 (ARMv8 / AArch64)                      |
| Family/Model         | ARMv8 Family 8 Model D49 (Cortex-X1 class)  |
| Manufacturer         | Microsoft Corporation                        |
| Physical sockets     | 1                                            |
| Physical cores       | 4                                            |
| Logical CPUs         | 4 (no SMT / Hyper-Threading)                 |
| Max clock speed      | 3,399 MHz                                    |
| L2 cache             | 4 MB                                         |
| L3 cache             | 128 MB                                       |

Microsoft Cobalt 100 is a custom ARM64 server chip (comparable to Neoverse N2 / Cortex-X1 generation), used in Azure infrastructure.

---

## Memory (RAM)

| Property                  | Value        |
|---------------------------|--------------|
| Total physical RAM        | 15.99 GB     |
| Page file (commit limit)  | 2.88 GB      |
| Total addressable (RAM + PF) | 18.87 GB  |

---

## Storage

### Physical Disks

Two physical disks are visible to the VM:

| Disk | Model                        | Interface | Media | Size     | Partitions | Role              |
|------|------------------------------|-----------|-------|----------|------------|-------------------|
| 0    | Microsoft NVMe Direct Disk v2 | NVMe/SCSI | SSD   | 220.0 GB | **0 — completely unpartitioned** | Unassigned raw disk |
| 1    | Microsoft Virtual Disk        | SAS/SCSI  | Fixed | 256.0 GB | 4          | OS / boot disk    |

> **Disk 0 (NVMe)** is a raw, unpartitioned NVMe SSD. It has no volumes, no drive letter, and no filesystem — it is entirely unused raw storage accessible only through low-level APIs. Firmware revision: `NVMDV002`. Serial: `708B_3835_5C05_4C56_AA32_748D_948F_8EB1`.
>
> **Disk 1 (Virtual SAS)** is a Hyper-V virtual disk hosting the entire OS.

### Disk 1 Partition Layout (GPT)

| Partition | Type          | Size    | Offset  | Filesystem | Drive | Label    | Notes           |
|-----------|---------------|---------|---------|------------|-------|----------|-----------------|
| 1         | Reserved      | 16 MB   | 1 MB    | —          | —     | —        | GPT reserved    |
| 2         | Recovery      | 450 MB  | 17 MB   | NTFS       | —     | Recovery | Hidden, WinRE   |
| 3         | System (EFI)  | 99 MB   | 467 MB  | FAT32      | —     | —        | EFI boot files  |
| 4         | Primary (OS)  | 255.4 GB| 566 MB  | NTFS       | `C:`  | Windows  | Boot / OS volume |

### C: Volume (NTFS)

| Property                      | Value                      |
|-------------------------------|----------------------------|
| Volume label                  | Windows                    |
| Filesystem                    | NTFS 3.1 (LFS 2.0)         |
| Volume serial number          | `FA1AA1B7`                 |
| Cluster size                  | 4,096 bytes (4 KB)         |
| Bytes per sector              | 512                        |
| Bytes per physical sector     | 4,096                      |
| Total clusters                | 66,963,963                 |
| Total capacity                | 274,284,392,448 bytes (255.45 GB) |
| MFT valid data length         | ~1,012 MB                  |
| MFT zone size                 | ~200 MB                    |
| Total reserved (NTFS)         | ~378 MB                    |
| Volume storage reserve        | ~274 MB                    |
| **Free at session start**     | **~119–123 GB** (varies by session) |

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

**Method:** Python 3.13 (ARM64) benchmark using `concurrent.futures.ProcessPoolExecutor` (process-based) and `threading.Thread` (thread-based). Each worker executes 10 million floating-point `sqrt` iterations. Serial baseline measured first.

**Environment:** Python 3.13.11 (ARM64), `os.cpu_count() = 4`.

#### Process-based scaling (true parallelism — no GIL)

| Workers (N) | Wall time | Throughput speedup | Efficiency |
|-------------|-----------|-------------------|------------|
| 1           | 0.899 s   | 0.82×             | 82%        |
| 2           | 0.831 s   | 1.77×             | 88%        |
| 3           | 0.846 s   | 2.61×             | 87%        |
| 4           | 1.183 s   | 2.49×             | 62%        |

> Serial baseline: **0.736 s** (single-threaded, no pool overhead).
>
> The 1-worker pool case is slower than serial due to `ProcessPoolExecutor` process-spawn overhead (~0.16 s). With N=3 workers, near-linear 2.61× speedup is achieved. N=4 drops to 2.49× because on this ARM64 platform all 4 logical CPUs are physical cores (no SMT) — contention for shared cache/memory bus at full load causes the slight regression. Process pool startup overhead is also more noticeable at small workloads.

#### Thread-based scaling (GIL-limited for CPU-bound Python code)

| Threads (N) | Wall time | Relative to serial |
|-------------|-----------|-------------------|
| 1           | 0.740 s   | 1.0× (baseline)   |
| 2           | 1.489 s   | 2.0× **slower**   |
| 3           | 2.229 s   | 3.0× **slower**   |
| 4           | 2.980 s   | 4.1× **slower**   |

Python threads scale **linearly worse** with thread count for CPU-bound work due to the **Global Interpreter Lock (GIL)**. The GIL allows only one thread to execute Python bytecode at a time, so N threads each contend for the lock and total wall time grows as N × serial_time.

**Conclusion:**
- **4 physical cores** are available, each independent (no SMT/HyperThreading).
- **True CPU parallelism** requires `multiprocessing` or `ProcessPoolExecutor` (bypasses GIL).
- Best observed speedup: **2.61× with 3 processes** (87% efficiency).
- **Python threads do not provide CPU parallelism** for CPU-bound workloads on any Python version with the GIL.
- For I/O-bound workloads, threads are fully effective (GIL is released during I/O).

---

### RAM Test — Allocation to Exhaustion

**Method:** Python `bytearray` allocations with every 4 KB page touched to force physical mapping. `GlobalMemoryStatusEx` (Win32 API) used to report available RAM and page-file usage after each allocation. Allocation was freed between steps (`del buf; gc.collect()`).

**Starting state:** 15.99 GB physical RAM, 2.88 GB page file, 12.86 GB available at test start.

| Allocated | Outcome | Avail RAM after | Avail PF after | Notes |
|-----------|---------|-----------------|----------------|-------|
| 1 GB      | ✅ OK   | 11.59 GB        | 14.49 GB       |       |
| 4 GB      | ✅ OK   | 8.07 GB         | 11.49 GB       |       |
| 8 GB      | ✅ OK   | 4.72 GB         | 7.48 GB        |       |
| 12 GB     | ✅ OK   | 0.80 GB         | 3.46 GB        |       |
| 13 GB     | ✅ OK   | 0.42 GB         | 2.46 GB        | ⚠ RAM nearly full — spilling to PF |
| 14 GB     | ✅ OK   | 0.49 GB         | 1.51 GB        | Majority in page file |
| 15 GB     | ✅ OK   | 0.97 GB         | 0.43 GB        | Page file nearly exhausted |
| 16 GB     | ✅ OK   | 0.41 GB         | 0.46 GB        | Beyond physical RAM ceiling |
| 17 GB     | ✅ OK   | 2.13 GB         | 0.49 GB        | OS reclaims cache aggressively |
| 18 GB     | ✅ OK   | 0.36 GB         | 0.47 GB        | Near total virtual limit |
| 19 GB     | ✅ OK   | 0.23 GB         | 0.49 GB        | Beyond total RAM+PF advertised |

**Last successful allocation: 19.00 GB** (no `MemoryError` was triggered up to 19 GB).

**Observations:**
- Allocations beyond ~13 GB begin to use the page file heavily.
- The OS reports total virtual memory as 18.87 GB (RAM + PF), yet 19 GB was allocatable — this is because the OS compresses and reclaims in-use pages, and the advertised page-file figure is conservative.
- Available RAM oscillates between attempts as the OS kernel aggressively moves cache to swap.
- No artificial per-process memory cap was detected.

**Conclusion:** The full ~16 GB physical RAM plus ~3 GB page file is accessible. Memory allocations beyond the hardware total succeed because Windows compresses pages and aggressively swaps caches. **There is no hidden per-process memory cap below the physical hardware ceiling.**

---

### Disk Test — Full Fill Capacity & Write Speed

**Method:** Write 1 GB files sequentially to `C:\Users\...\AppData\Local\Temp\disk_fill_test\`, `fsync` after each file. Stop when ≤ 2 GB remains (safety margin to keep the system functional). Chunk size reduced adaptively as free space shrinks.

**Starting state:** 255.4 GB total, 132.7 GB used, **122.75 GB free**.

#### Fill progress (every 10 GB milestone)

| Written total | Free after  | Write speed | Notes                       |
|---------------|-------------|-------------|-----------------------------|
| 10 GB         | 110.5 GB    | ~120 MB/s   |                             |
| 20 GB         | 97.5 GB     | ~150 MB/s   |                             |
| 30 GB         | 89.1 GB     | ~117 MB/s   |                             |
| 40 GB         | 79.1 GB     | ~158 MB/s   |                             |
| 50 GB         | 69.1 GB     | ~116 MB/s   |                             |
| 60 GB         | 59.1 GB     | ~154 MB/s   |                             |
| 70 GB         | 49.1 GB     | ~154 MB/s   |                             |
| 80 GB         | 39.1 GB     | ~116 MB/s   |                             |
| 90 GB         | 29.1 GB     | ~153 MB/s   |                             |
| 100 GB        | 19.1 GB     | ~156 MB/s   |                             |
| 110 GB        | 9.1 GB      | ~156 MB/s   |                             |
| 114 GB        | 5.1 GB      | ~160 MB/s   | switched to 256 MB chunks   |
| 117.09 GB     | **2.00 GB** | **~165 MB/s** | ← **safety margin hit — stopped** |

> Write speed was **consistently 86–165 MB/s** (typical ~115–160 MB/s) from the very first gigabyte all the way to 117 GB — no degradation at any fill level.

#### Final state

| Metric                        | Value         |
|-------------------------------|---------------|
| Initial free space            | 122.75 GB     |
| **Total successfully written**| **117.09 GB** |
| Files created                 | 139           |
| Free at stop (safety margin)  | 2.00 GB       |
| Free after cleanup            | 118.8 GB      |
| Write speed range             | 86–165 MB/s   |
| Errors / quota events         | None          |

#### Conclusions

- **117.09 GB** was written without any `OSError`, quota error, or throttling.
- Write speed was **completely stable** from first to last byte — no NVMe saturation or caching artifact was observed.
- The full ~122.75 GB of free disk space is entirely usable; the only limit is the raw volume capacity.
- Cleanup (`rmdir /s`) restored the full free space instantly.

---

## Summary — Key Differences vs Previous Linux/AMD64 Environment

| Dimension       | Previous (Linux/AMD64)                   | Current (Windows/ARM64)               |
|-----------------|------------------------------------------|---------------------------------------|
| OS              | Ubuntu (Linux)                           | Windows 10/11 Enterprise ARM64        |
| CPU             | AMD EPYC 7763 (x86_64)                   | Microsoft Cobalt 100 (ARM64)          |
| Logical CPUs    | 4 (2 physical × 2 HT threads)           | 4 (4 physical, no SMT)               |
| CPU topology    | 2 cores × 2 HT threads (real HT)        | 4 independent physical cores          |
| CPU parallelism | Near-linear (3.91× at N=4)              | Up to 2.61× at N=3 (pool overhead)   |
| Total RAM       | 15 GiB + 3 GiB swap                     | 15.99 GB + 2.88 GB page file          |
| Max allocatable | ~16.9 GB (OOM-kill at 17 GB)            | ≥19 GB (no MemoryError found)         |
| Disk device     | `/dev/root` ext4 SSD (91 GB free)       | `C:` NTFS NVMe SSD (~122.75 GB free)  |
| Write speed     | ~330 MB/s                                | ~120–155 MB/s                         |
| Session timeout | 59 minutes                               | 59 minutes (unchanged)                |
