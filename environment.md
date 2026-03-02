# Coding Agent Environment

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

### CPU Test — Multiprocessing & Threading

**Method:** Python `multiprocessing` and `threading` modules used to launch one worker per logical CPU, each burning CPU for 1 second.

**Results:**

| Test                        | Wall-clock time | Conclusion                                         |
|-----------------------------|----------------|----------------------------------------------------|
| 4 processes (one per CPU)   | **1.01 s**     | ✅ True parallel execution — all 4 ran concurrently |
| 4 threads (one per CPU)     | **1.08 s**     | ✅ OS-level parallelism confirmed (Python GIL note below) |

- `/proc/cpuinfo` confirms **2 unique physical core IDs** (`core id 0` and `core id 1`), each with **2 sibling threads** = 4 logical CPUs.
- Multiprocessing wall time of ~1.01 s for 4 × 1 s burn tasks confirms that all 4 logical CPUs are genuinely available and run in parallel.
- Python threads: the GIL limits true CPU-bound parallelism in Python threads, but the OS still scheduled them across cores (wall ≈ 1.08 s rather than 4 s).

**Conclusion:** The `lscpu` report is accurate. We have **2 physical cores × 2 threads/core = 4 logical CPUs**, all usable simultaneously.

---

### RAM Tests — Gradual Allocation

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

**Conclusion:** The full ~15 GiB RAM (plus 3 GiB swap) is accessible. There is **no hidden 8 GB RAM barrier**.

---

### Disk Tests — Multiple Small Files & Single Large File

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

**Conclusion:** The root filesystem (`/dev/root`) truly has ~91 GB free at session start. There is **no hidden 14 GB disk limit**.
