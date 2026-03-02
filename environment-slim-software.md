# Coding Agent Software Environment — Slim Container (Current)

> **This document covers the current "slim" container software environment.**  
> The previous Ubuntu/AMD64 runner software environment is documented in **[`environment-ubuntu-amd64-software.md`](./environment-ubuntu-amd64-software.md)**.

This document is a factual inventory of software **pre-installed and available out of the box** in the slim GitHub Copilot Coding Agent container session.

> **Hardware & OS environment:** See [`environment-slim.md`](./environment-slim.md)  
> **Previous (Ubuntu AMD64) software environment:** See [`environment-ubuntu-amd64-software.md`](./environment-ubuntu-amd64-software.md)

---

## Quick-Reference — What Is Available

| Category | Tool / Runtime | Version | In PATH | Notes |
|---|---|---|---|---|
| Shell | Bash | 5.2.21 | ✅ | Default shell |
| VCS | Git | 2.52.0 | ✅ | — |
| VCS | GitHub CLI (`gh`) | 2.85.0 | ✅ | — |
| VCS | Mercurial (`hg`) | 6.7.2 | ✅ | — |
| Python | CPython 3.12.3 | 3.12.3 | ✅ (default `python3`) | x86_64; **no toolcache versions** |
| Node.js | Node.js | 24.13.0 | ✅ | x64; **only one version** |
| Node.js | npm | 11.6.2 | ✅ | Bundled with Node 24 |
| Node.js | corepack | 0.34.5 | ✅ | — |
| C/C++ | GCC | 13.3.0 | ✅ | Ubuntu build; `x86_64-linux-gnu` |
| C/C++ | GNU Make | 4.3 | ✅ | `/usr/bin/make` |
| Containers | Docker | 29.1.5 | ✅ | — |
| Cloud | AWS CLI v2 | 2.33.2 | ✅ | — |
| Cloud | Azure CLI (`az`) | 2.82.0 | ✅ | — |
| Utility | curl | 8.5.0 | ✅ | OpenSSL/3.0.13 |
| Utility | 7-Zip (`7z`) | 23.01 | ✅ | — |
| Utility | tar | 1.35 | ✅ | GNU tar |
| Utility | unzip | 6.00 | ✅ | — |
| Utility | jq | 1.7 | ✅ | — |
| Utility | OpenSSL | 3.0.13 | ✅ | — |
| Scripting | Perl | 5.38.2 | ✅ | x86_64-linux-gnu-thread-multi |

---

## Python

**Default (`python3`):** Python **3.12.3** (system package, `/usr/bin/python3`).

> **No toolcache versions** — unlike the full Ubuntu AMD64 runner, there is no `/opt/hostedtoolcache/Python/` directory. Only the system Python 3.12.3 is available. No PyPy versions are installed.

**pip 24.0** is pre-installed (system). Notable pre-installed pip packages (system-wide):

| Package | Version |
|---|---|
| pip | 24.0 |
| pipx | 1.8.0 |
| argcomplete | 3.6.3 |
| click | 8.3.1 |
| cryptography | 41.0.7 |
| packaging | 25.0 |
| setuptools | 68.1.2 |
| mercurial | 6.7.2 |

`python -m venv` works out of the box.

> **Not available:** `boto3`, `botocore`, `Jinja2`, `PyYAML`, `requests`, `rich`, `ansible`, `cloud-init` — none of these are pre-installed, unlike in the full runner.

---

## Node.js

Node.js **v24.13.0** is the default. In PATH from `/opt/hostedtoolcache/node/24.13.0/x64/`.

> **Only one version** — unlike the full Ubuntu AMD64 runner (which also has Node 22 and 20 in the toolcache), only Node 24.13.0 is present.

**npm 11.6.2** is bundled with Node 24.

Global npm packages pre-installed:

| Package | Version |
|---|---|
| corepack | 0.34.5 |
| npm | 11.6.2 |

`yarn`, `pnpm`, `deno`, and `bun` are **not** pre-installed.

---

## C and C++

| Tool | Version | In PATH | Notes |
|---|---|---|---|
| GCC | 13.3.0 | ✅ | Ubuntu build; `x86_64-linux-gnu` |
| GNU Make | 4.3 | ✅ | `/usr/bin/make` |

> **Not available:** Clang/LLVM, CMake, Ninja, vcpkg — none are present in the slim container.

---

## Version Control

| Tool | Version | Notes |
|---|---|---|
| Git | 2.52.0 | — |
| GitHub CLI (`gh`) | 2.85.0 | — |
| Mercurial (`hg`) | 6.7.2 | — |

---

## Containers

| Tool | Version | Notes |
|---|---|---|
| Docker | 29.1.5 | ✅ Available |

> **Podman is not available** in the slim container (unlike the full Ubuntu AMD64 runner which has both Docker and Podman).

---

## Cloud CLIs

| Tool | Version | Notes |
|---|---|---|
| AWS CLI v2 | 2.33.2 | — |
| Azure CLI (`az`) | 2.82.0 | — |

> **Not available:** Pulumi, Ansible — absent from the slim container.

---

## General Utilities

| Tool | Version | Notes |
|---|---|---|
| curl | 8.5.0 | OpenSSL/3.0.13 |
| 7-Zip (`7z`) | 23.01 | — |
| tar | 1.35 | GNU tar |
| unzip | 6.00 | — |
| jq | 1.7 | — |
| OpenSSL | 3.0.13 | — |
| Perl | 5.38.2 | `/usr/bin/perl` |

---

## Package Managers

| Manager | Version | Notes |
|---|---|---|
| pip | 24.0 | Python packages |
| npm | 11.6.2 | Node packages |

`apt` (Ubuntu) is available for installing additional system packages.

---

## Key Differences vs Ubuntu AMD64 Runner

| Category | Ubuntu AMD64 (Previous) | Slim Container (Current) |
|---|---|---|
| Python default | 3.12.3 | 3.12.3 ✅ (same) |
| Python toolcache | 3.10–3.14 + PyPy 3.9/3.10/3.11 | ❌ None |
| Node.js | 24.13.1 + Node 22, 20 in toolcache | 24.13.0 (one version only) |
| .NET | 10.0.102 (default), 8.x, 9.x | ❌ Not installed |
| Java | Temurin 17 (default), 8, 11, 21, 25 | ❌ Not installed |
| Go | 1.24.13 + 1.22/1.23/1.25 in toolcache | ❌ Not installed |
| Rust | 1.93.1 stable | ❌ Not installed |
| Ruby | 3.2.3 (system) + toolcache | ❌ Not installed |
| PHP | 8.3.6 | ❌ Not installed |
| Haskell GHC | 9.14.1 | ❌ Not installed |
| Swift | 6.2.3 | ❌ Not installed |
| Clang / LLVM | 18.1.3 | ❌ Not installed |
| CMake / Ninja | 3.31.6 / 1.13.2 | ❌ Not installed |
| vcpkg | 2025-12-16 | ❌ Not installed |
| Kubernetes (kubectl, helm, kind, minikube) | ✅ All present | ❌ None |
| Podman | 4.9.3 ✅ | ❌ Not available |
| Pulumi | v3.223.0 | ❌ Not installed |
| Ansible | 2.20.3 | ❌ Not installed |
| yarn | 1.22.22 | ❌ Not installed |
| boto3/botocore | Pre-installed in pip | ❌ Not pre-installed |
| Jinja2 / PyYAML | Pre-installed in pip | ❌ Not pre-installed |
