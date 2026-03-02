# Coding Agent Environment — Slim Container (Current)

> **This document covers the current "slim" container environment.**  
> The previous Ubuntu/AMD64 runner environment is documented in **[`environment-ubuntu-amd64.md`](./environment-ubuntu-amd64.md)**.

This document describes the hardware and runtime constraints of the "slim" GitHub Copilot Coding Agent container environment.

> **Software environment** (languages, compilers, tools): See [`environment-slim-software.md`](./environment-slim-software.md).  
> **Previous (Ubuntu AMD64) environment:** See [`environment-ubuntu-amd64.md`](./environment-ubuntu-amd64.md).

---

## Platform

| Property        | Value                                                             |
|-----------------|-------------------------------------------------------------------|
| OS              | Ubuntu 24.04.3 LTS (Noble Numbat)                                 |
| Kernel          | 6.1.146.1-microsoft-standard                                      |
| Runner label    | Slim container (GitHub-hosted)                                    |
| Virtualisation  | Microsoft Hyper-V / Azure (fully virtualised); container overlay  |
| Container type  | Docker / GCS overlay container                                    |
| Hypervisor      | Microsoft                                                         |

> **Key difference from the Ubuntu AMD64 runner:** This is a *slim* containerised environment. It runs on the same underlying AMD EPYC host hardware but exposes only **1 vCPU** (vs 4 logical CPUs), **5 GiB RAM** (vs 15 GiB), and **50 GB** of overlay storage (vs 150 GB physical disk), with **no swap**. The software set is also significantly reduced — see [`environment-slim-software.md`](./environment-slim-software.md).

---

## CPU

| Property             | Value                              |
|----------------------|------------------------------------|
| Architecture         | x86_64                             |
| Model                | AMD EPYC 7763 64-Core Processor    |
| CPU family           | 25 (Zen 3 / Milan)                 |
| Hypervisor           | Microsoft (fully virtualised)      |
| Sockets              | 1                                  |
| Physical cores       | 1                                  |
| Threads per core     | 1 (no SMT / no Hyper-Threading)    |
| Logical CPUs (total) | **1**                              |
| Clock speed          | ~2445 MHz                          |
| L1d / L1i cache      | 32 KiB (1 instance each)           |
| L2 cache             | 512 KiB (1 instance)               |
| L3 cache             | 32 MiB (shared, 1 instance)        |

`/proc/cpuinfo` reports a single processor (CPU 0), `core id 0`, `cpu cores: 1`, `siblings: 1` — **no Hyper-Threading**.

---

## Memory (RAM)

| Type  | Total       | Used at start | Free at start | Available  | cgroup limit |
|-------|-------------|---------------|---------------|------------|--------------|
| RAM   | ~4.94 GiB   | ~0.97 GiB     | ~3.79 GiB     | ~3.85 GiB  | **5 GiB**    |
| Swap  | 0           | 0             | 0             | —          | —            |

> The cgroup memory limit (`memory.limit_in_bytes`) is exactly **5,368,709,120 bytes (5 GiB)**. There is **no swap**. The OS itself consumes ~1.1 GiB at session start, leaving ~3.85 GiB available to user workloads.

---

## Storage

### Filesystem Layout

The root filesystem is a Docker **overlay** filesystem backed by SCSI virtual disks mounted via the GitHub Container Service (GCS) runtime:

| Filesystem  | Type    | Total | Used at start | Free at start | Mount Point |
|-------------|---------|-------|---------------|---------------|-------------|
| `overlay`   | overlay | 50 GB | ~2.4 GB       | **~44–45 GB** | `/` (root)  |
| `tmpfs`     | tmpfs   | 64 MB | 0             | 64 MB         | `/dev`      |
| `shm`       | tmpfs   | 64 MB | 0             | 64 MB         | `/dev/shm`  |
| `/dev/sdc`  | ext4    | 50 GB | ~2.4 GB       | ~45 GB        | `/etc/hosts`, `/etc/hostname`, `/etc/resolv.conf` |

The overlay lower directories are SCSI block devices (`/run/mounts/scsi/m6`, `m8`, `m9`). The upper/work directories are on a scratch volume within the GCS container runtime.

> **No raw / unpartitioned disks are present** — only the overlay filesystem and container-injected SCSI devices used for bind-mounted config files. Unlike the Ubuntu AMD64 runner, there is no separate physical disk to format or fill beyond the overlay root.

---

## Timeouts

| Timeout                            | Value      | Source                             |
|------------------------------------|------------|------------------------------------|
| Coding Agent maximum session time  | 59 minutes | `COPILOT_AGENT_TIMEOUT_MIN=59` env var |

---

## Empirical Tests

---

### Physical Disk Survey — Overlay Filesystem Only

**Objective:** Determine the physical disk layout and whether any raw or unmounted disks are available.

**Method:** `lsblk`, `df -h`, `/proc/mounts` inspection.

**Result:** No traditional block device partitioning is visible. The root filesystem is an **overlay** backed by multiple read-only lower SCSI layers and a writable upper layer (GCS scratch volume). Several small SCSI virtual disks (`sda`–`sdk`) are visible but are used for container image layers and bind-mount injection — they are not separately usable. The sole writable space is via the overlay root (50 GB total, ~44–45 GB free at start).

| Device    | Size   | Notes                                  |
|-----------|--------|----------------------------------------|
| `sda`     | 73.2 MB| Container image layer (read-only)      |
| `sdb`     | 784 KB | Container image layer (read-only)      |
| `sdc`     | 50 GB  | Bind-mount disk (`/etc/hosts`, etc.)   |
| `sdd–sdg` | 2–4 MB | Container image layers (read-only)     |
| `sdh–sdj` | 64 KB  | Container image layers (read-only)     |
| `sdk`     | 5.4 GB | Container image layer (read-only)      |
| `overlay` | 50 GB  | **Root filesystem — writable**         |

**Conclusion:** This environment has **no separate physical disk** to initialise or format. All writable storage is provided through the overlay root filesystem.

---

### Disk Test — Fill Capacity & Write Speed

**Objective:** Verify the true amount of writable disk space and measure sequential write throughput.

**Method:** Write 1 GB files sequentially to `/tmp/disk_fill_test/` with `fsync` after each file, stopping when ≤ 2 GB free. Python 3.12.3.

**Starting state:** ~44.37 GB free on overlay root.

| Written total | Free after  | Write speed |
|---------------|-------------|-------------|
| 5 GB          | 39.37 GB    | 1,632 MB/s  |
| 10 GB         | 34.37 GB    | 1,600 MB/s  |
| 15 GB         | 29.37 GB    | 1,647 MB/s  |
| 20 GB         | 24.37 GB    | 1,565 MB/s  |
| 25 GB         | 19.37 GB    | 1,478 MB/s  |
| 30 GB         | 14.37 GB    | 1,437 MB/s  |
| 35 GB         | 9.37 GB     | 1,550 MB/s  |
| 40 GB         | 4.37 GB     | 1,173 MB/s  |
| 42 GB         | 2.37 GB     | 1,514 MB/s  |
| **43 GB (stopped)** | **~1.4 GB** | — |

| Metric                         | Value              |
|--------------------------------|--------------------|
| Initial free space             | ~44.37 GB          |
| **Total successfully written** | **43 GB**          |
| Avg write speed                | **~1,527 MB/s**    |
| Min write speed                | ~345 MB/s (last file, near full) |
| Max write speed                | ~1,884 MB/s        |
| Errors / quota events          | None               |
| Free after cleanup             | ~44 GB restored    |

**Conclusion:** The overlay filesystem delivers **exceptional write throughput** (~1,500 MB/s average), far exceeding the physical-disk-backed Ubuntu AMD64 runner (~342 MB/s). This is consistent with the GCS scratch volume being backed by fast NVMe storage optimised for container workloads. Total writable capacity is ~44 GB (approximately the advertised 45 GB free minus filesystem overhead and stop margin).

---

### CPU Test — Single vCPU Throughput

**Note:** Only **1 logical CPU** is allocated to this container. Parallelism tests are not applicable.

#### Python benchmark (serial, 20M sqrt iterations)

**Method:** Python 3.12.3 pure-Python loop: `x += math.sqrt(float(i+1))` × 20,000,000.

| Metric | Value |
|--------|-------|
| 20M ops (Python 3.12) | 3.57 s |
| **Serial throughput** | **5.59 Mops/s** |

> Python throughput is approximately **3.3× lower** than on the full Ubuntu AMD64 runner (18.74 Mops/s). The C benchmark below shows only ~1.3× slower hardware, suggesting the Python difference is primarily due to reduced cache efficiency in the containerised overlay environment and single-CPU resource allocation.

#### C benchmark (serial, 300M sqrt iterations)

**Method:** GCC-compiled C binary (`gcc -O2 -march=native`), 300 million `sqrt` operations, timed with `clock_gettime(CLOCK_MONOTONIC)`.

| Metric | Value |
|--------|-------|
| 300M ops (C, single CPU) | 1.09 s |
| **C throughput** | **275 Mops/s** |

> The full Ubuntu AMD64 runner achieves **363 Mops/s** under the same test (1.32× faster). The ~25% difference reflects fewer allocated vCPUs sharing the physical core resources rather than a fundamentally different CPU.

#### Multi-process scaling

With only 1 logical CPU, `ProcessPoolExecutor` provides no benefit — workers queue serially:

| Workers (N) | Wall time | Effective throughput | Notes |
|-------------|-----------|----------------------|-------|
| 1 (baseline)| 1.09 s (C)| 275 Mops/s           | Full single-vCPU throughput |
| 2+          | N × 1.09 s| 275 Mops/s (no gain) | Serialised — single vCPU |

**Conclusion:** This environment provides **exactly 1 vCPU** for all compute. No parallelism is available. All concurrent processes or threads share the single logical CPU time-slice.

---

### RAM Test — Allocation Scaling

**Method:** Python `bytearray` allocations with every 4 KB page touched to force physical mapping. `/proc/meminfo` polled after each allocation. Each allocation freed before the next.

**Starting state:** ~4.94 GiB RAM, no swap, cgroup limit 5 GiB, ~3.85 GiB available.

| Allocated | Outcome | Avail RAM after | Swap used | Notes |
|-----------|---------|-----------------|-----------|-------|
| 0.5 GB    | ✅ OK   | 3,340 MB        | 0 MB      |       |
| 1.0 GB    | ✅ OK   | 2,823 MB        | 0 MB      |       |
| 1.5 GB    | ✅ OK   | 2,316 MB        | 0 MB      |       |
| 2.0 GB    | ✅ OK   | 1,803 MB        | 0 MB      |       |
| 2.5 GB    | ✅ OK   | 1,290 MB        | 0 MB      |       |
| 3.0 GB    | ✅ OK   |   781 MB        | 0 MB      |       |
| 3.5 GB    | ✅ OK   |   267 MB        | 0 MB      | Near cgroup limit |
| 4.0 GB    | ❌ **OOM-killed** | — | — | cgroup OOM-killer fired (exit 137) |

**Conclusion:** The cgroup limit of **5 GiB** is the hard ceiling. With ~1.1 GiB consumed by the OS and Python process at session start, the maximum single allocatable block is approximately **3.5 GB**. There is **no swap** — once RAM is exhausted the cgroup OOM-killer fires immediately. This is a significantly more constrained memory environment than the Ubuntu AMD64 runner (15 GiB RAM + 3 GiB swap, with OOM only at ~17 GB total allocation).

---

## Summary — Comparison to Ubuntu AMD64 Environment

| Dimension           | Ubuntu AMD64 (Previous)                            | Slim Container (Current)                          |
|---------------------|----------------------------------------------------|---------------------------------------------------|
| OS                  | Ubuntu 24.04.3 LTS                                 | Ubuntu 24.04.3 LTS                                |
| Kernel              | 6.14.0-1017-azure                                  | 6.1.146.1-microsoft-standard                      |
| Runner type         | Full GitHub-hosted VM                              | Slim container (GCS overlay)                      |
| CPU model           | AMD EPYC 7763 (x86_64)                             | AMD EPYC 7763 (x86_64)                            |
| Logical CPUs        | 4 (2 physical × 2 HT)                             | **1 (no HT)**                                     |
| Compute parallelism | ProcessPool: 1.67× at N=4                          | **None — single vCPU**                            |
| C sqrt throughput   | 363 Mops/s                                        | 275 Mops/s (~1.3× slower)                         |
| Python sqrt throughput | 18.74 Mops/s                                   | 5.59 Mops/s (~3.3× slower)                        |
| Total RAM           | 15 GiB + 3 GiB swap                               | **~4.94 GiB, no swap** (cgroup limit: 5 GiB)      |
| Max single allocation | ~16 GB (RAM+swap)                               | **~3.5 GB**                                       |
| Primary disk        | `/dev/root` ext4, 145 GB, ~91 GB free             | overlay (GCS), 50 GB, ~44 GB free                 |
| Write speed         | ~342 MB/s                                         | **~1,527 MB/s avg** (fast overlay/NVMe)           |
| Raw/extra disk      | ❌ None                                           | ❌ None                                            |
| Docker / containers | ✅ Docker 28 + Podman 4.9                         | ✅ Docker 29 (Podman **not available**)            |
| Software richness   | Full toolchain (Go, Rust, Java, .NET, Swift, …)   | **Minimal** (Python, Node, GCC, Git, Docker only) |
| Session timeout     | 59 minutes                                        | 59 minutes (unchanged)                            |
