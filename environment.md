# Coding Agent Environment — Linux AMD64 (Session 1–4 Archive)

> **This document archives Sessions 1–4 of the Linux/AMD64 environment.**
> The current Ubuntu/AMD64 environment is documented in **[`environment-ubuntu-amd64.md`](./environment-ubuntu-amd64.md)**.  
> The current Windows/ARM64 environment is documented in **[`environment-windows-arm.md`](./environment-windows-arm.md)**.

This document describes the hardware and timeout constraints of the environment in which the GitHub Copilot Coding Agent runs.

---

## Storage

| Device     | Total Size | Used  | Available | Mount Point |
|------------|-----------|-------|-----------|-------------|
| /dev/sda   | 150 GB    | —     | —         | (physical disk) |
| /dev/root  | 145 GB    | 54 GB | 91 GB     | `/` (root filesystem) |
| /dev/sda16 | 881 MB    | 63 MB | 757 MB    | `/boot` |
| /dev/sda15 | 105 MB    | 6.2 MB| 99 MB     | `/boot/efi` |

---

## CPU

| Property             | Value                              |
|----------------------|------------------------------------|
| Architecture         | x86_64                             |
| Model                | AMD EPYC 7763 64-Core Processor    |
| Hypervisor           | Microsoft (fully virtualised)      |
| Sockets              | 1                                  |
| Physical cores       | 2 (2 cores per socket)             |
| Threads per core     | 2 (SMT / Hyper-Threading enabled)  |
| Logical CPUs (total) | 4                                  |

---

## Memory (RAM)

| Type  | Total  | Used   | Free   | Available |
|-------|--------|--------|--------|-----------|
| RAM   | 15 GiB | 2.2 GiB| 10 GiB | 13 GiB    |
| Swap  | 3.0 GiB| 0 B    | 3.0 GiB| —         |

---

## Timeouts

| Timeout                            | Value      | Source                             |
|------------------------------------|------------|------------------------------------|
| Coding Agent maximum session time  | 59 minutes | `COPILOT_AGENT_TIMEOUT_MIN=59` env var |

The `COPILOT_AGENT_TIMEOUT_MIN` environment variable is set to **59**, meaning the Copilot Coding Agent will be terminated after a maximum of **59 minutes** per session. There are no additional system-level CPU or process-time limits (`ulimit -t` is unlimited).

---

## Empirical Tests

The sections below document the results of stress tests run to validate (or refute) hypotheses about enforced limits on CPU, RAM, and disk.

---

### CPU Test — Multiprocessing & Threading (Session 1)

**Method:** Python `multiprocessing` and `threading` modules used to launch one worker per logical CPU, each burning CPU for 1 second.

**Results:**

| Test                        | Wall-clock time | Conclusion                                         |
|-----------------------------|----------------|----------------------------------------------------|
| 4 processes (one per CPU)   | **1.01 s**     | ✅ True parallel execution — all 4 ran concurrently |
| 4 threads (one per CPU)     | **1.08 s**     | ✅ OS-level parallelism confirmed (Python GIL note below) |

- `/proc/cpuinfo` confirms **2 unique physical core IDs** (`core id 0` and `core id 1`), each with **2 sibling threads** = 4 logical CPUs.
- Multiprocessing wall time of ~1.01 s for 4 × 1 s burn tasks confirms that all 4 logical CPUs are genuinely available and run in parallel.
- Python threads: the GIL limits true CPU-bound parallelism in Python threads, but the OS still scheduled them across cores (wall ≈ 1.08 s rather than 4 s).

**Conclusion (Session 1):** The `lscpu` report is accurate. We have **2 physical cores × 2 threads/core = 4 logical CPUs**, all usable simultaneously.

---

### CPU Test — Deep Performance Characterisation (Session 2)

To avoid Python's GIL and measure true hardware behaviour, a **C binary** (compiled with `gcc -O2 -march=native`) was used for all CPU benchmarks. Tests were run with CPU affinity pinning via `taskset`.

#### 1. Throughput Scaling (C benchmark, no GIL)

Each worker executes 200 million floating-point `sqrt` operations. N workers are launched simultaneously. The key metric is **throughput speedup** = (N × serial time) / wall-clock time.

| Workers (N) | Avg wall time | Throughput speedup | Efficiency |
|-------------|--------------|-------------------|-----------|
| 1           | 1.653 s      | 1.00×             | 100%      |
| 2           | 1.656 s      | **2.00×**         | 100%      |
| 3           | 1.685 s      | **2.94×**         | 98%       |
| 4           | 1.692 s      | **3.91×**         | 98%       |

Wall-clock time is nearly constant from N=1 to N=4 (only +2.3%). All 4 workers run simultaneously — no queuing. Throughput scales near-linearly.

#### 2. CPU Affinity Pinning — Same-Core vs Different-Core

Workers were pinned to specific logical CPUs using `taskset`. `/proc/cpuinfo` reports CPU 0 & 1 share physical core 0, and CPU 2 & 3 share physical core 1.

**Compute-bound test (FP sqrt, 300M iterations):**

| Configuration                            | Wall time | Overhead vs solo |
|------------------------------------------|----------|-----------------|
| 1 worker on CPU 0 (baseline)             | 2.479 s  | —               |
| 2 workers: CPU 0 + CPU 1 (same phys core)| 2.528 s  | **+2.0%**       |
| 2 workers: CPU 0 + CPU 2 (diff phys core)| 2.482 s  | **+0.1%**       |
| 4 workers: CPU 0+1+2+3 (all)             | 2.550 s  | **+2.9%**       |

Only 2% overhead even when two workers share the same physical core. With true Hyper-Threading one typically sees 30–50% slowdown for compute-bound work on a shared core. This indicates the **hypervisor allocates each vCPU as an independent execution unit**.

**Memory bandwidth test (32 MB working-set, cache-stride reads):**

| Configuration                            | Bandwidth    | Ratio vs solo |
|------------------------------------------|-------------|--------------|
| 1 worker on CPU 0                        | 30.93 GB/s  | 1.000        |
| 2 workers: CPU 0 + CPU 1 (same phys core)| 22.10 GB/s each | **0.715** |
| 2 workers: CPU 0 + CPU 2 (diff phys core)| 26.16 GB/s each | 0.846    |

Memory bandwidth shows moderate degradation (28%) when two workers are assigned to the same physical core's logical CPUs, compared to 15% degradation across different cores. This is the only measurable sign of resource sharing — likely L2/L3 cache and memory-controller contention — consistent with real HT architecture at the memory subsystem level.

#### Summary — True CPU Topology

| Characteristic | Observed |
|---------------|---------|
| Logical CPUs | **4** |
| Compute-bound throughput scaling | **Near-linear up to 4** (3.91× at N=4) |
| Same-core compute overhead | **2%** (vs ~30–50% expected for real HT) |
| Same-core memory-BW penalty | **28%** (moderate — cache sharing present) |
| Different-core memory-BW penalty | **15%** (DRAM bandwidth contention) |

**Conclusion:** The 4 logical CPUs behave as **near-independent execution units** for compute-bound work. The hypervisor presents the two HT threads of each physical core as distinct, non-competing vCPUs. Memory-bandwidth-bound workloads do see modest contention between HT siblings, which is the only evidence of the underlying 2-core × 2-HT topology. For practical purposes, **all 4 vCPUs deliver full independent compute throughput simultaneously**.

---

### RAM Tests — Gradual Allocation (Session 1)

Allocations were done by creating a `bytearray` and touching every page (4 KB stride) to force actual physical memory mapping.

| Test size | Outcome | RAM used after alloc | Swap used after alloc | Notes |
|-----------|---------|---------------------|-----------------------|-------|
| **1 GB**  | ✅ Success | ~3.0 GB | 0 MB | No swap pressure |
| **4 GB**  | ✅ Success | ~6.0 GB | 0 MB | No swap pressure |
| **8 GB**  | ✅ Success | ~10.2 GB | 0 MB | **8 GB barrier hypothesis REFUTED** |
| **12 GB** | ✅ Success | ~14.2 GB | 0 MB | Only ~223 MB RAM + 1.7 GB available headroom left |
| **14 GB** | ✅ Success | ~15.8 GB | **~445 MB** | Spills into swap; still succeeds |
| **15 GB** | ✅ Success | ~15.8 GB | **~1,367 MB** | Heavy swap use; succeeds |

**Hypothesis tested:** An 8 GB hard ceiling on usable RAM.
**Result: REFUTED.** No enforced memory limit was found. 8, 12, 14, and even 15 GB allocations all succeeded. Allocations above ~13 GB begin to spill into the 3 GB swap partition, but no hard cap or `MemoryError` was triggered.

**Conclusion (Session 1):** The full ~15 GiB RAM (plus 3 GiB swap) is accessible. There is **no hidden 8 GB RAM barrier**.

---

### RAM Tests — Exhaustion to OOM (Session 2)

Testing continued beyond 15 GB to find the exact point where the Linux OOM-killer fires. Each test started from a clean swap state (`swapoff -a && swapon -a`). Pages were touched at 4 KB stride to guarantee physical allocation. Total addressable memory = **15,990 MB RAM + 3,071 MB swap = 19,061 MB (~18.6 GB)**.

| Test size | Outcome | Swap used / free | Notes |
|-----------|---------|-----------------|-------|
| **16.0 GB** | ✅ Success | 2,253 MB used / 818 MB free | —    |
| **16.5 GB** | ✅ Success | 2,710 MB used / 361 MB free | —    |
| **16.75 GB**| ✅ Success | 3,011 MB used / **60 MB free** | Nearly full swap |
| **16.9 GB** | ✅ Success | **3,071 MB used / 0 MB free** | **Swap 100% exhausted** |
| **17.0 GB** | ❌ **OOM-killed** (exit 137 / SIGKILL) | — | Kernel OOM-killer fired |

**`dmesg` evidence for 17 GB OOM:**
```
python3 invoked oom-killer: gfp_mask=0x140cca(GFP_HIGHUSER_MOVABLE|__GFP_COMP)
Out of memory: Killed process (python3) total-vm:17845648kB, anon-rss:15282544kB
```

**Gradual memory pressure observations (16.5 GB test, 256 MB reporting intervals):**
- RAM stays at ~160–380 MB available throughout (kernel aggressively swaps out caches)
- Swap usage climbs steadily from ~750 MB → 1,200 MB → 2,700 MB as allocation fills RAM
- No sudden cutoff or artificial throttle — swap fills linearly

**Conclusion:** The practical maximum allocatable memory is approximately **16.9 GB** (RAM 15.99 GB + Swap 3.07 GB = 19.06 GB total, minus ~2.16 GB for OS + process overhead). Attempting 17.0 GB triggers the Linux OOM-killer. There is no artificial limit below the hardware ceiling — the full RAM+swap is usable up to the true physical boundary.

---

### Disk Tests — Multiple Small Files & Single Large File (Session 1)

#### Multiple files (100 MB each)

**Method:** Write N × 100 MB files to `/tmp`, check after each group.

| Files written | Total size | Outcome | Available disk after |
|---------------|-----------|---------|----------------------|
| 10 files      | 1,000 MB  | ✅ Success | ~90 GB |
| 150 files     | 15,000 MB (~15 GB) | ✅ Success | ~76 GB |

No errors at any point. Written across individual files, the 14 GB barrier hypothesis was not triggered.

#### Single large file

**Method:** Write a single file in 1 GB increments up to 20 GB, `fsync` after each increment.

| Checkpoint | Cumulative size | Outcome | Available disk |
|------------|----------------|---------|---------------|
| 1 GB       | 1 GB  | ✅ | ~90 GB |
| 5 GB       | 5 GB  | ✅ | ~86 GB |
| 10 GB      | 10 GB | ✅ | ~81 GB |
| 14 GB      | 14 GB | ✅ | **~77 GB** — **14 GB barrier hypothesis REFUTED** |
| 20 GB      | 20 GB | ✅ | ~71 GB |

**Hypothesis tested:** Only ~14 GB of disk space is actually usable.
**Result: REFUTED.** A single 20 GB file was written successfully without any error. `df` confirms that 71 GB of the 91 GB advertised free space remained after the 20 GB write. All space appears genuinely accessible.

**Conclusion (Session 1):** The root filesystem (`/dev/root`) truly has ~91 GB free at session start. There is **no hidden 14 GB disk limit**.

---

### Disk Test — Full Exhaustion (Session 2)

**Method:** Write 1 GB files sequentially to `/tmp/disk_fill/` until only 500 MB remains (safety margin to keep the system functional). Each file was `fsync`'d after writing.

**Starting state:** 91.4 GB available on `/dev/root`.

| Milestone         | Files written | Used disk | Available | Write speed |
|-------------------|--------------|----------|-----------|------------|
| 10 GB written     | File 10      | 62.9 GB  | 81.4 GB   | ~330 MB/s  |
| 20 GB written     | File 20      | 72.9 GB  | 71.4 GB   | ~331 MB/s  |
| 30 GB written     | File 30      | 82.9 GB  | 61.4 GB   | ~331 MB/s  |
| 40 GB written     | File 40      | 92.9 GB  | 51.4 GB   | ~330 MB/s  |
| 50 GB written     | File 50      | 102.9 GB | 41.4 GB   | ~331 MB/s  |
| 60 GB written     | File 60      | 112.9 GB | 31.4 GB   | ~331 MB/s  |
| 70 GB written     | File 70      | 122.9 GB | 21.4 GB   | ~331 MB/s  |
| 80 GB written     | File 80      | 132.9 GB | 11.4 GB   | ~331 MB/s  |
| 90 GB written     | File 90      | 142.9 GB | 1.4 GB    | ~331 MB/s  |
| 90.86 GB (**stopped**) | File 91 | 143.8 GB | **0.5 GB** | ~354 MB/s |

**Observations:**
- Total written: **90.86 GB** (99.5% of available space used)
- Write speed was **consistently ~330 MB/s** throughout the entire fill — no slowdown at any point
- No `OSError` or throttling was observed at any fill level
- After cleanup, disk returned to 91.4 GB available — no permanent impact

**Conclusion:** The full ~91.4 GB of free disk space on `/dev/root` is **entirely usable**. Write performance remains constant from empty to near-full. There are **no artificial quotas, no hidden disk limits, and no speed throttling** at any fill level.

---

### Disk Test — True ENOSPC Exhaustion (Session 4)

**Goal:** Fill all user-available space to the very last byte and document the exact ENOSPC boundary.

**Method:** Write files to `/tmp/disk_exhaust/` with adaptive chunk sizes as space shrinks — switching from 1 GB → 100 MB → 10 MB → 1 MB → 128 KB → 4 KB chunks as `f_bavail` (user-available blocks) decreases. After ENOSPC, probe with decreasing sizes (4 KB → 1 B) to confirm the hard boundary. All files `fsync`'d after each write.

#### Filesystem layout

| Metric | Value |
|--------|-------|
| Filesystem total size | 144.26 GB |
| In use at test start | 52.89 GB |
| **User-available at start** (`df` Avail) | **91.3529 GB** (98,089,472,000 bytes) |
| Root-reserved blocks | 4,096 blocks = **16 MB** (0.01% of total) |
| Inodes at start | 19,529,728 total / 18,554,858 free |

> `df` already subtracts the 16 MB root-reserved blocks; the **91.35 GB reported by `df` is the true user-available space**.

#### Fill progress (every 10 GB)

| Written total | User-avail remaining | Chunk size |
|--------------|---------------------|-----------|
| 10 GB        | 81.35 GB            | 1 GB       |
| 20 GB        | 71.35 GB            | 1 GB       |
| 30 GB        | 61.35 GB            | 1 GB       |
| 40 GB        | 51.35 GB            | 1 GB       |
| 50 GB        | 41.35 GB            | 1 GB       |
| 60 GB        | 31.35 GB            | 1 GB       |
| 70 GB        | 21.35 GB            | 1 GB       |
| 80 GB        | 11.35 GB            | 1 GB       |
| 90 GB        | 1.34 GB             | 100 MB     |
| 91.16 GB     | 191 MB              | 1 MB       |
| 91.35 GB     | ~4 KB               | 4 KB       |
| **91.3509 GB** | **0 bytes** | ❌ **ENOSPC** |

#### ENOSPC details

| Metric | Value |
|--------|-------|
| Total written before ENOSPC | **91.3509 GB** |
| User-avail before final write | 4,096 bytes (1 block) |
| Chunk size attempted | 4,096 bytes |
| `errno` | **28 (ENOSPC)** |
| `f_bavail` at failure | **0** (no user blocks remain) |
| `f_bfree` at failure | 4,096 blocks = 16,777,216 bytes (root-reserved headroom intact) |
| Unwritable remainder | ~2 MB (filesystem metadata overhead for 1,000+ test files) |

#### Post-ENOSPC probe

After ENOSPC was triggered, attempts to write any amount of data as a non-root user all failed immediately:

| Probe size | Result |
|-----------|--------|
| 4,096 bytes | ❌ ENOSPC |
| 1,024 bytes | ❌ ENOSPC |
| 512 bytes   | ❌ ENOSPC |
| 64 bytes    | ❌ ENOSPC |
| 1 byte      | ❌ ENOSPC |

At the point of ENOSPC, `f_bavail = 0` — every single user-available block is consumed. The kernel will not grant even a 1-byte write. The 16 MB root-reserved space (`f_bfree - f_bavail`) remains intact and is inaccessible to non-root processes.

**After cleanup:** `rm -rf /tmp/disk_exhaust` restored the full 91.3529 GB — no permanent impact.

#### Conclusion

The user-available disk space on `/dev/root` is exactly **91.35 GB** (`df` Avail). This is the hard ceiling for non-root writes. Key findings:

- The filesystem has **16 MB reserved for root** (`tune2fs` reserved-block-count = 4,096 blocks × 4 KB). This is already excluded from the `df` Avail figure.
- A user can consume **all 91.35 GB** — there is no hidden per-user quota or artificial sub-limit.
- ENOSPC fires when `f_bavail` (user free blocks) reaches **0**. The very last bytes are consumed by filesystem metadata (directory entries, inodes) for test files — approximately **2 MB** of the 91.35 GB is consumed by metadata when writing many small files near the boundary.
- Write speed remained **~330 MB/s** throughout the fill — no slowdown even within the last 100 MB.
