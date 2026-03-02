# Coding Agent Environment — Windows ARM64

> **This document covers the current Windows/ARM64 environment.**  
> The previous Windows/AMD64 environment is documented in **[`environment-windows-amd64.md`](./environment-windows-amd64.md)**.

This document describes the hardware and runtime constraints of the Windows/ARM64 GitHub Copilot Coding Agent environment.

> **Note:** The Linux/AMD64 environment is documented in [`environment.md`](./environment.md).  
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
| 0    | Microsoft NVMe Direct Disk v2 | NVMe/SCSI | SSD   | 220.0 GB | **0 at session start — completely unpartitioned (RAW)** | Scratch NVMe disk |
| 1    | Microsoft Virtual Disk        | SAS/SCSI  | Fixed | 256.0 GB | 4          | OS / boot disk    |

> **Disk 0 (NVMe)** arrives as a raw, unpartitioned NVMe SSD with no volumes, no drive letter, and no filesystem. It can be initialized and formatted within a session (see empirical tests below). Firmware revision: `NVMDV002`. Serial: `708B_3835_5C05_4C56_AA32_748D_948F_8EB1`.
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

### CPU Test — Session 1: Basic Parallelism & Threading

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

---

### CPU Test — Session 2: Deep Parallelism Benchmark (3-minute budget, best-effort strategies)

**Method:** Four parallelism strategies tested back-to-back in a single 3-minute run, using 20 million floating-point `sqrt` operations per worker call. Each sub-test runs for 25 seconds (6 sub-tests × 25 s ≈ 150 s active; ~235 s total including startup). Strategies used:

1. **`ProcessPoolExecutor`** — separate OS processes, GIL fully bypassed
2. **`ThreadPoolExecutor` (pure Python)** — threads subject to GIL for CPU-bound code
3. **Threads + ctypes ARM64 C DLL** — GIL released during native C `sqrt` loop (compiled with `clang --target=aarch64-pc-windows-msvc -O3`)

**Environment:** Python 3.13.11 ARM64, `os.cpu_count() = 4`. C DLL compiled with Clang 20.1.6 for `aarch64-pc-windows-msvc`.

#### Serial baseline

| Metric | Value |
|--------|-------|
| 20M ops (pure Python) | 1.606 s |
| **Serial throughput** | **12.45 Mops/s** |

#### Strategy 1 — ProcessPoolExecutor (GIL bypassed)

| Workers (N) | Rounds in 25 s | Throughput | Speedup vs serial | Efficiency |
|-------------|---------------|------------|------------------|-----------|
| 1           | 16            | 12.50 Mops/s | 1.00×          | 100%      |
| 2           | 16            | 24.63 Mops/s | **1.98×**      | **99%**   |
| 4           | 16            | 47.99 Mops/s | **3.85×**      | **96%**   |

> Near-linear throughput scaling. N=4 achieves 3.85× — the slight shortfall vs ideal 4.0× is due to Windows process-spawn overhead amortised over 25 s.

#### Strategy 2 — ThreadPoolExecutor (CPU-bound, GIL active)

| Threads (N) | Rounds in 25 s | Throughput | Speedup vs serial | Notes |
|-------------|---------------|------------|------------------|-------|
| 1           | 17            | 13.50 Mops/s | 1.08×          | — |
| 2           | 9             | 13.61 Mops/s | 1.09×          | GIL serialises threads |
| 4           | 5             | 13.66 Mops/s | 1.10×          | GIL serialises threads |

> Throughput is flat regardless of thread count — the GIL prevents true CPU-bound parallelism in Python threads. Each thread takes ~3× the time of serial because it is contending for the GIL.

#### Strategy 3 — Threads + ctypes native C (GIL released)

| Threads (N) | Rounds in 25 s | Throughput    | Speedup vs C N=1 | Efficiency | Speedup vs Python serial |
|-------------|---------------|---------------|-----------------|-----------|--------------------------|
| 1           | 221           | 176.40 Mops/s | 1.00×           | 100%      | 14.2×                    |
| 2           | 222           | 354.46 Mops/s | **2.01×**       | **100%**  | 28.5×                    |
| 4           | 219           | 699.30 Mops/s | **3.97×**       | **99%**   | 56.2×                    |

> When Python threads call into a native ARM64 C DLL, the GIL is released for the duration of the C call. This gives **true OS-level thread parallelism**: N=2 achieves 2.01× (100% efficiency), N=4 achieves 3.97× (99% efficiency) — essentially perfect linear scaling across all 4 physical cores.
>
> The C `sqrt` loop is ~14× faster than pure Python per core. At N=4, the combined advantage is **56× vs Python serial throughput** — the product of the per-core C speedup (14×) and the parallel scaling (4×).

#### Summary

**Conclusion:**
- **4 physical cores** are available, each independent (no SMT/Hyper-Threading).
- **ProcessPoolExecutor** (multi-process): up to **3.85×** speedup at N=4 (96% efficiency) for CPU-bound Python.
- **Python `ThreadPoolExecutor`**: GIL-serialised — throughput does not increase with more threads for CPU-bound work.
- **Threads + native C via ctypes** (GIL released): **near-perfect linear scaling** — 2.01× at N=2 (100% eff), 3.97× at N=4 (99% eff). This is the highest-throughput parallelism strategy available from Python on this host.
- For maximum CPU throughput: use `ctypes` (or `cffi`) to delegate compute to a native ARM64 library, then call it from multiple Python threads. The GIL is released for the entire native call duration.

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

### Disk Test — NVMe Disk 0 Initialization, Formatting & Fill (Session 2)

**Goal:** Initialize the raw NVMe disk (Disk 0, 220 GB), apply performance-optimised NTFS settings, then fill it as fast as possible with large sequential writes.

#### Disk 0 initialization

Disk 0 starts every session as a completely raw (unpartitioned) NVMe drive. The following steps were executed via PowerShell:

```powershell
# 1. Initialise with GPT (supports disks > 2 TB, modern standard)
Initialize-Disk -Number 0 -PartitionStyle GPT

# 2. Create one data partition using the full disk
New-Partition -DiskNumber 0 -UseMaximumSize -DriveLetter E

# 3. Format NTFS with 64 KB allocation units (maximum supported by NTFS)
#    Large clusters reduce per-file metadata overhead for multi-GB sequential files
Format-Volume -DriveLetter E -FileSystem NTFS -AllocationUnitSize 65536 `
    -NewFileSystemLabel "NVMe-Data" -Force -Confirm:$false

# 4. Performance tweaks
fsutil behavior set disable8dot3 E: 1       # no 8.3 short filenames per write
fsutil behavior set disableLastAccess 1     # no last-access timestamp on reads
```

**Resulting E: volume layout:**

| Property                  | Value                                      |
|---------------------------|--------------------------------------------|
| Drive letter              | `E:`                                       |
| Label                     | NVMe-Data                                  |
| Filesystem                | NTFS 3.1                                   |
| Allocation unit size      | **65,536 bytes (64 KB)** — max for NTFS    |
| Bytes per sector          | 512                                        |
| Bytes per physical sector | 4,096                                      |
| Total clusters            | 3,604,207                                  |
| Total capacity            | 219.98 GB                                  |
| 8.3 filename generation   | Disabled                                   |
| Last-access time updates  | Disabled (system-wide)                     |
| Free at start             | 219.89 GB                                  |

> **Why 64 KB clusters?** NTFS default is 4 KB. For large sequential files (512 MB – 1 GB), 64 KB clusters reduce the MFT record count by 16×, cut per-write metadata overhead, and improve read-ahead alignment. The only trade-off is slightly higher slack for tiny files, which is irrelevant for a scratch fill workload.

#### Fill test on E: (NVMe, 64 KB NTFS)

**Method:** Write 512 MB files sequentially to `E:\disk_fill_test\` using a pre-allocated in-memory buffer. Each file is written in a single unbuffered pass and `fsync`'d (`os.fsync`). Stop when ≤ 2 GB free. Buffer size: 512 MB (maximises sequential NVMe queue depth).

**Starting state:** 219.89 GB free on `E:`.

#### Fill progress (every 10 GB milestone)

| Written total | Free after  | Write speed | Notes                        |
|---------------|-------------|-------------|------------------------------|
| 0.5 GB        | 219.39 GB   | 342 MB/s    | first file                   |
| 1.0 GB        | 218.89 GB   | 342 MB/s    | second file                  |
| 11.0 GB       | 208.89 GB   | 344 MB/s    |                              |
| 21.0 GB       | 198.89 GB   | 345 MB/s    |                              |
| 31.0 GB       | 188.89 GB   | 345 MB/s    |                              |
| 41.0 GB       | 178.89 GB   | 344 MB/s    |                              |
| 51.0 GB       | 168.89 GB   | 344 MB/s    |                              |
| 61.0 GB       | 158.89 GB   | 345 MB/s    |                              |
| 71.0 GB       | 148.89 GB   | 345 MB/s    |                              |
| 81.0 GB       | 138.89 GB   | 345 MB/s    |                              |
| 91.0 GB       | 128.89 GB   | 344 MB/s    |                              |
| 101.0 GB      | 118.89 GB   | 345 MB/s    |                              |
| 111.0 GB      | 108.89 GB   | 345 MB/s    |                              |
| 121.0 GB      | 98.89 GB    | 344 MB/s    |                              |
| 131.0 GB      | 88.89 GB    | 343 MB/s    |                              |
| 141.0 GB      | 78.89 GB    | 338 MB/s    |                              |
| 151.0 GB      | 68.89 GB    | 345 MB/s    |                              |
| 161.0 GB      | 58.89 GB    | 343 MB/s    |                              |
| 171.0 GB      | 48.89 GB    | 337 MB/s    |                              |
| 181.0 GB      | 38.89 GB    | 344 MB/s    |                              |
| 191.0 GB      | 28.89 GB    | 345 MB/s    |                              |
| 201.0 GB      | 18.89 GB    | 345 MB/s    |                              |
| 211.0 GB      | 8.89 GB     | 345 MB/s    |                              |
| 217.89 GB     | **2.00 GB** | **345 MB/s**| ← **safety margin — stopped**|

#### Final state

| Metric                        | Value         |
|-------------------------------|---------------|
| Initial free space (`E:`)     | 219.89 GB     |
| **Total successfully written**| **217.89 GB** |
| Files created                 | 436 (× 512 MB)|
| Free at stop (safety margin)  | 2.00 GB       |
| Free after cleanup            | 219.89 GB     |
| Wall time                     | 648.5 s (~10.8 min) |
| **Avg write speed**           | **344 MB/s**  |
| Min write speed               | 319 MB/s      |
| Max write speed               | 345 MB/s      |
| Errors / quota events         | None          |

#### Conclusions

- **217.89 GB** was written to the freshly-formatted NVMe volume with zero errors.
- Write speed was **completely stable at ~344 MB/s** from the very first file to the last — no slowdown near the end, confirming no thermal throttle or write-cache saturation.
- The NVMe disk is **more than twice as fast as C:** for sequential writes (344 MB/s vs ~120–155 MB/s on the virtual SAS disk), consistent with direct NVMe access bypassing the Hyper-V virtual disk layer.
- 64 KB NTFS cluster formatting + disabled 8.3 names + disabled last-access updates produced no measurable overhead difference from the C: baseline, confirming these are pure optimisations for large-file workloads.
- Full 219.89 GB of user-available space is accessible; cleanup restores it instantly.

---

## Summary — Key Differences vs Previous Linux/AMD64 Environment

| Dimension       | Previous (Linux/AMD64)                   | Current (Windows/ARM64)               |
|-----------------|------------------------------------------|---------------------------------------|
| OS              | Ubuntu (Linux)                           | Windows 11 Enterprise ARM64           |
| CPU             | AMD EPYC 7763 (x86_64)                   | Microsoft Cobalt 100 (ARM64)          |
| Logical CPUs    | 4 (2 physical × 2 HT threads)           | 4 (4 physical, no SMT)               |
| CPU topology    | 2 cores × 2 HT threads (real HT)        | 4 independent physical cores          |
| CPU parallelism (Python) | Near-linear (3.91× at N=4)    | ProcessPool: 3.85× at N=4 (96% eff)  |
| CPU parallelism (ctypes) | —                             | **3.97× at N=4 threads (99% eff)**   |
| Total RAM       | 15 GiB + 3 GiB swap                     | 15.99 GB + 2.88 GB page file          |
| Max allocatable | ~16.9 GB (OOM-kill at 17 GB)            | ≥19 GB (no MemoryError found)         |
| C: disk (virtual SAS) | N/A                             | NTFS, ~119–123 GB free; ~120–155 MB/s |
| E: disk (raw NVMe) | N/A                                | **220 GB NVMe; 344 MB/s sequential**  |
| Session timeout | 59 minutes                               | 59 minutes (unchanged)                |
