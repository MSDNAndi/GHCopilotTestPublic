# Coding Agent Environment — Ubuntu AMD64 (Previous)

> **This document covers the previous Ubuntu/AMD64 runner environment.**  
> The current "slim" container environment is documented in **[`environment-slim.md`](./environment-slim.md)**.  
> The previous Windows/ARM64 environment is documented in **[`environment-windows-arm.md`](./environment-windows-arm.md)**.

This document describes the hardware and runtime constraints of the Ubuntu/AMD64 GitHub Copilot Coding Agent environment.

> **Software environment** (languages, compilers, tools): See [`environment-ubuntu-amd64-software.md`](./environment-ubuntu-amd64-software.md).  
> **Original Linux/AMD64 session data** (Sessions 1–4): See [`environment.md`](./environment.md).

---

## Platform

| Property        | Value                                              |
|-----------------|----------------------------------------------------|
| OS              | Ubuntu 24.04.3 LTS (Noble Numbat)                  |
| Kernel          | 6.14.0-1017-azure                                  |
| Runner label    | `ubuntu-latest` / `ubuntu-24.04`                   |
| Virtualisation  | Microsoft Hyper-V / Azure (fully virtualised)      |
| Hypervisor      | Microsoft                                          |

---

## CPU

| Property             | Value                              |
|----------------------|------------------------------------|
| Architecture         | x86_64                             |
| Model                | AMD EPYC 7763 64-Core Processor    |
| CPU family           | 25 (Zen 3 / Milan)                 |
| Hypervisor           | Microsoft (fully virtualised)      |
| Sockets              | 1                                  |
| Physical cores       | 2 (2 cores per socket)             |
| Threads per core     | 2 (SMT / Hyper-Threading enabled)  |
| Logical CPUs (total) | 4                                  |
| L1d / L1i cache      | 64 KiB per core (2 instances)      |
| L2 cache             | 1 MiB per core (2 instances)       |
| L3 cache             | 32 MiB (shared, 1 instance)        |

`/proc/cpuinfo` reports:
- CPU 0 & 1 → `core id 0` (physical core 0, HT pair)
- CPU 2 & 3 → `core id 1` (physical core 1, HT pair)

---

## Memory (RAM)

| Type  | Total   | Used at start | Free at start | Available |
|-------|---------|---------------|---------------|-----------|
| RAM   | 15 GiB  | 1.5 GiB       | 11 GiB        | 14 GiB    |
| Swap  | 3.0 GiB | 0 B           | 3.0 GiB       | —         |

---

## Storage

### Physical Disks

Only one physical disk is visible to the VM:

| Device   | Model        | Size   | Type      | Partitions | Status                    |
|----------|--------------|--------|-----------|------------|---------------------------|
| `/dev/sda` | Virtual Disk | 150 GB | Rotational (virtual) | 4 | Fully partitioned — no raw disk |

> **No raw / unpartitioned / unmounted physical disk is present in this environment.** Unlike the Windows ARM64 runner (which had a second raw NVMe disk, Disk 0, available for initialisation), the Ubuntu AMD64 runner provides only a single pre-partitioned virtual disk. There is nothing to additionally mount or format.

### Partition Layout (GPT)

| Device      | Size   | Filesystem | Mount Point | Notes               |
|-------------|--------|------------|-------------|---------------------|
| `/dev/sda`  | 150 GB | —          | —           | GPT disk            |
| `/dev/sda1` | 149 GB | ext4       | `/` (root)  | Main filesystem     |
| `/dev/sda14`|   4 MB | —          | —           | BIOS boot           |
| `/dev/sda15`| 106 MB | vfat       | `/boot/efi` | EFI System          |
| `/dev/sda16`| 913 MB | ext4       | `/boot`     | Extended boot (ABL) |

### Root Filesystem (`/dev/root` → `/dev/sda1`)

| Property                 | Value                         |
|--------------------------|-------------------------------|
| Filesystem               | ext4                          |
| Total size               | 145 GB                        |
| Used at session start    | ~54 GB                        |
| **Free at session start**| **~91 GB** (90.49 GB observed)|
| Mount point              | `/` (root)                    |

---

## Timeouts

| Timeout                            | Value      | Source                             |
|------------------------------------|------------|------------------------------------|
| Coding Agent maximum session time  | 59 minutes | `COPILOT_AGENT_TIMEOUT_MIN=59` env var |

---

## Empirical Tests

---

### Physical Disk Survey — No Unmounted Disks Found

**Objective:** Determine whether any raw or unmounted physical disks are available (as was the case on Windows ARM64, which had a 220 GB raw NVMe disk).

**Method:** `lsblk`, `fdisk -l`, `/proc/partitions` enumeration.

**Result:** Only `/dev/sda` (150 GB virtual disk) is present. It arrives fully partitioned with four GPT partitions covering the entire disk. There are **no raw, unpartitioned, or unmounted additional disks**.

**Conclusion:** Unlike the Windows ARM64 environment (where Disk 0 was a pristine 220 GB NVMe SSD ready to be initialised), the Ubuntu AMD64 runner provides a single pre-configured OS disk. No additional disk initialisation, formatting, or fill benchmark is possible.

---

### CPU Test — Parallelism & Threading

**Method:** Python `concurrent.futures.ProcessPoolExecutor` (process-based) and `threading.Thread` (thread-based). Each worker executes 20 million floating-point `sqrt` iterations. Serial baseline measured first. Python 3.12.3 (x86_64).

#### Serial baseline

| Metric | Value |
|--------|-------|
| 20M ops (Python 3.12) | 1.067 s |
| **Serial throughput** | **18.74 Mops/s** |

#### Process-based scaling (no GIL)

| Workers (N) | Wall time | Throughput   | Speedup vs serial | Efficiency |
|-------------|-----------|-------------|------------------|-----------|
| 1           | 1.133 s   | 17.64 Mops/s | 0.94× (pool overhead) | — |
| 2           | 1.379 s   | 29.00 Mops/s | **1.55×**         | 77%        |
| 4           | 2.563 s   | 31.22 Mops/s | **1.67×**         | 42%        |

> N=2 achieves 1.55× — decent, but below the theoretical 2×. N=4 reaches only 1.67× despite 4 logical CPUs. This is consistent with **2 physical cores × 2 HT threads**: HT pairs share execution resources, so all 4 logical CPUs cannot independently sustain compute-bound Python work at full throughput simultaneously.

#### Thread-based scaling (GIL active)

| Threads (N) | Wall time | Throughput   | vs serial |
|-------------|-----------|-------------|-----------|
| 1           | 1.086 s   | 18.41 Mops/s | ≈1.0×    |
| 2           | 4.003 s   |  9.99 Mops/s | 0.53× **slower** |
| 4           | 4.512 s   | 17.73 Mops/s | ≈1.0× |

> Python threads serialise CPU-bound work via the GIL. N=2 shows heavy GIL contention (each thread stalls waiting for the lock); N=4 returns near-serial throughput because the 4 HT threads time-slice through the single GIL token.

---

### CPU Test — Deep Characterisation (C Benchmark, Affinity Pinning)

To bypass the Python GIL and measure true hardware behaviour, a C binary (`gcc -O2 -march=native`) was used with `taskset` CPU pinning. Test: 300 million `sqrt` operations per worker.

#### Throughput by configuration

| Configuration                              | Wall time each | Throughput per worker | vs solo   |
|--------------------------------------------|----------------|-----------------------|-----------|
| 1 worker on CPU 0 (baseline)               | 0.826 s        | **363 Mops/s**        | 1.00×     |
| 2 workers: CPU 0 + CPU 1 (same phys core)  | ~1.66 s each   | 181 Mops/s each       | **−50%**  |
| 2 workers: CPU 0 + CPU 2 (diff phys cores) | ~0.82 s each   | ~364 Mops/s each      | ≈ 0%      |
| 4 workers: CPU 0+1+2+3 (all)               | ~1.65 s each   | ~181 Mops/s each      | **−50%**  |

#### Interpretation

Same-physical-core pairs (CPU 0 & 1, or CPU 2 & 3) share the actual execution pipeline of one AMD EPYC physical core. When both HT threads run compute-bound `sqrt` in parallel, they contend for the shared FPU/SIMD units and the effective throughput per thread drops by ~50% — the textbook Hyper-Threading slowdown for compute-bound workloads.

Cross-core pairs (CPU 0 & 2) each have their own dedicated physical core resources. Wall time is essentially unchanged versus solo, confirming no inter-core contention for this workload.

| Characteristic | Observed |
|----------------|---------|
| Physical cores | **2** |
| HT threads per core | **2** (logical CPUs 0+1 and 2+3) |
| Compute-bound same-core overhead | **~50%** (textbook HT behaviour) |
| Compute-bound cross-core overhead | **< 1%** (fully independent) |
| Max independent compute throughput | **2 physical cores** simultaneously |

**Conclusion:** The 4 logical CPUs are **2 physical cores with Hyper-Threading**. For compute-bound C work, the effective independent parallelism is **2×** (one per physical core), not 4×. Placing two workers on different physical cores (e.g. CPU 0 and CPU 2) gives near-perfect independent throughput.

---

### RAM Test — Allocation Scaling

**Method:** Python `bytearray` allocations with every 4 KB page touched to force physical mapping. `/proc/meminfo` polled after each allocation.

**Starting state:** 15 GiB RAM, 3 GiB swap, ~14 GiB available.

| Allocated | Outcome | Avail RAM after | Swap used after | Notes |
|-----------|---------|-----------------|-----------------|-------|
| 1 GB      | ✅ OK   | 13,529 MB       | 0 MB            |       |
| 4 GB      | ✅ OK   | 10,447 MB       | 0 MB            |       |
| 8 GB      | ✅ OK   |  6,343 MB       | 0 MB            |       |
| 12 GB     | ✅ OK   |  2,229 MB       | 0 MB            |       |
| 14 GB     | ✅ OK   |    427 MB       | 216 MB          | Spilling into swap |
| 15 GB     | ✅ OK   |     87 MB       | 817 MB          | Heavy swap pressure |
| 16 GB     | ✅ OK   |     79 MB       | 1,867 MB        | Beyond physical RAM; OS compresses |

**Conclusion:** All tested allocations up to 16 GB succeeded. The pattern matches the previous session data (see `environment.md`): the full 15 GiB RAM plus ~3 GiB swap is accessible with no artificial per-process cap.

---

### Disk Test — Fill Capacity & Write Speed

**Method:** Write 1 GB files sequentially to `/tmp/disk_fill_test/` with `fsync` after each, stopping when ≤ 2 GB free.

**Starting state:** 90.49 GB free on `/dev/root`.

| Written total | Free after  | Write speed |
|---------------|-------------|-------------|
| 10 GB         | 81.49 GB    | 343 MB/s    |
| 20 GB         | 71.49 GB    | 343 MB/s    |
| 30 GB         | 61.49 GB    | 343 MB/s    |
| 40 GB         | 51.49 GB    | 342 MB/s    |
| 50 GB         | 41.49 GB    | 341 MB/s    |
| 60 GB         | 31.49 GB    | 342 MB/s    |
| 70 GB         | 21.49 GB    | 341 MB/s    |
| 80 GB         | 11.49 GB    | 347 MB/s    |
| **88 GB (stopped)** | **2.00 GB** | — |

| Metric                       | Value         |
|------------------------------|---------------|
| Initial free space           | 90.49 GB      |
| **Total successfully written** | **88 GB**   |
| Avg write speed              | **342 MB/s**  |
| Min write speed              | 316 MB/s      |
| Max write speed              | 347 MB/s      |
| Errors / quota events        | None          |
| Free after cleanup           | 90.49 GB restored |

**Conclusion:** The full ~90 GB of available space is accessible. Write speed is consistently **~342 MB/s** — stable from first to last file with no throughput degradation. There are no artificial disk quotas or hidden limits.

---

## Summary — Comparison to Windows ARM64 Environment

| Dimension           | Windows ARM64 (Previous)                           | Ubuntu AMD64 (Current)                    |
|---------------------|----------------------------------------------------|-------------------------------------------|
| OS                  | Windows 11 Enterprise ARM64                        | Ubuntu 24.04.3 LTS                        |
| CPU                 | Microsoft Cobalt 100 (ARM64, 4 physical cores)     | AMD EPYC 7763 (x86_64, 2 cores × 2 HT)   |
| Logical CPUs        | 4 (no SMT)                                         | 4 (2 physical × 2 HT)                    |
| Compute parallelism | ProcessPool: 3.85× at N=4 (96%)                   | ProcessPool: 1.67× at N=4 (HT limited)   |
| Same-core HT penalty| N/A (physical cores, no HT)                        | **~50%** (classic HT FPU contention)     |
| Total RAM           | 15.99 GB + 2.88 GB page file                       | 15 GiB + 3 GiB swap                      |
| Primary disk (free) | C: ~119–123 GB; ~120–155 MB/s write                | `/dev/root` ~91 GB; ~342 MB/s write       |
| Raw/extra disk      | ✅ 220 GB NVMe (raw, initialised in session)       | ❌ None — single pre-partitioned disk     |
| Docker / containers | ❌ Not available                                   | ✅ Docker 28.0.4 + Podman 4.9.3 available |
| Session timeout     | 59 minutes                                         | 59 minutes (unchanged)                    |
