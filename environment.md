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
