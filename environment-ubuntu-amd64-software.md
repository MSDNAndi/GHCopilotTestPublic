# Coding Agent Software Environment — Ubuntu AMD64 (Previous)

> **This document covers the previous Ubuntu/AMD64 runner software environment.**  
> The current "slim" container software environment is documented in **[`environment-slim-software.md`](./environment-slim-software.md)**.  
> The previous Windows/ARM64 software environment is documented in **[`environment-windows-arm-software.md`](./environment-windows-arm-software.md)**.

This document is a factual inventory of software **pre-installed and available out of the box** in the Ubuntu/AMD64 GitHub Copilot Coding Agent session.

> **Hardware & OS environment:** See [`environment-ubuntu-amd64.md`](./environment-ubuntu-amd64.md)  
> **Previous (Windows/ARM64) environment:** See [`environment-windows-arm.md`](./environment-windows-arm.md)

---

## Quick-Reference — What Is Available

| Category | Tool / Runtime | Version | In PATH | Notes |
|---|---|---|---|---|
| Shell | Bash | 5.2.21 | ✅ | Default shell |
| VCS | Git | 2.53.0 | ✅ | LFS enabled |
| VCS | GitHub CLI (`gh`) | 2.87.3 | ✅ | — |
| VCS | Mercurial (`hg`) | 6.7.2 | ✅ | — |
| Python | CPython 3.12.3 | 3.12.3 | ✅ (default `python3`) | x86_64 |
| Python | CPython 3.10.19–3.14.3 | various | via toolcache | See table below |
| Python | PyPy 3.9.19 / 3.10.16 / 3.11.13 | various | via toolcache | — |
| Node.js | Node.js | 24.13.1 | ✅ | x64 |
| Node.js | yarn | 1.22.22 | ✅ | — |
| Node.js | corepack | 0.34.6 | ✅ | — |
| .NET | .NET SDK | 10.0.102 (default) | ✅ | x64 |
| .NET | .NET SDK | 8.x, 9.x | ✅ | — |
| Java | Eclipse Temurin JDK | 17.0.18 LTS | ✅ | x64 |
| Java | Maven | 3.9.12 | ✅ | — |
| Java | Gradle | 9.3.1 | ✅ | — |
| Java | Kotlin | 2.3.10 | ✅ | — |
| Go | Go | 1.24.13 | ✅ | x64 |
| Rust | rustc / cargo | 1.93.1 stable | ✅ | x64 |
| Ruby | Ruby + gem | 3.2.3 / 3.4.20 | ✅ | x64 |
| PHP | PHP | 8.3.6 | ✅ | x64 |
| Perl | Perl | 5.38.2 | ✅ | x64 |
| Haskell | GHC | 9.14.1 | ✅ | x64 |
| Swift | Swift | 6.2.3 | ✅ | x64 |
| C/C++ | GCC | 13.3.0 | ✅ | x64 |
| C/C++ | Clang / LLVM | 18.1.3 | ✅ | x64 |
| C/C++ | CMake | 3.31.6 | ✅ | — |
| C/C++ | Ninja | 1.13.2 | ✅ | — |
| C/C++ | GNU Make | 4.3 | ✅ | — |
| C/C++ | vcpkg | 2025-12-16 | ✅ | — |
| Containers | Docker | 28.0.4 | ✅ | — |
| Containers | Podman | 4.9.3 | ✅ | — |
| Kubernetes | kubectl | 1.35.1 | ✅ | — |
| Kubernetes | Helm | 3.20.0 | ✅ | — |
| Kubernetes | kind | 0.31.0 | ✅ | — |
| Kubernetes | minikube | 1.38.1 | ✅ | — |
| Cloud | AWS CLI v2 | 2.33.28 | ✅ | — |
| Cloud | Azure CLI (`az`) | 2.83.0 | ✅ | — |
| Cloud | Pulumi | v3.223.0 | ✅ | — |
| Cloud | Ansible | 2.20.3 | ✅ | — |
| Utility | curl | 8.5.0 | ✅ | OpenSSL/3.0.13 |
| Utility | 7-Zip (`7z`) | 23.01 | ✅ | — |
| Utility | tar | 1.35 | ✅ | GNU tar |
| Utility | unzip | 6.00 | ✅ | — |
| Utility | jq | 1.7 | ✅ | — |
| Utility | OpenSSL | 3.0.13 | ✅ | — |

---

## Python

**Default (`python3`):** Python **3.12.3** (system package, `/usr/bin/python3`).

Additional CPython versions are pre-installed in the toolcache at `/opt/hostedtoolcache/Python/`:

| Version | Status |
|---------|--------|
| **3.12.3** (system) | `python3` in PATH |
| 3.10.19 | toolcache |
| 3.11.14 | toolcache |
| 3.12.12 | toolcache |
| 3.13.12 | toolcache |
| 3.14.3  | toolcache |

PyPy versions in toolcache (`/opt/hostedtoolcache/PyPy/`):

| Version | Notes |
|---------|-------|
| PyPy 3.9.19  | — |
| PyPy 3.10.16 | — |
| PyPy 3.11.13 | — |

**pip 24.0** is pre-installed (system). Notable pre-installed pip packages (system-wide):

| Package | Version |
|---|---|
| pip | 24.0 |
| pipx | 1.8.0 |
| boto3 | 1.34.46 |
| botocore | 1.34.46 |
| Jinja2 | 3.1.2 |
| PyYAML | 6.0.1 |
| cryptography | 41.0.7 |
| requests | 2.31.0 |
| click | 8.1.6 |
| colorama | 0.4.6 |
| argcomplete | 3.6.3 |
| rich | 13.7.1 |
| packaging | 24.0 |
| setuptools | 68.1.2 |
| mercurial | 6.7.2 |
| ansible | (via pipx) |
| cloud-init | 25.2 |

`python -m venv` works out of the box.

---

## Node.js

Node.js **v24.13.1** is the default. In PATH from `/opt/hostedtoolcache/node/24.13.1/x64/`.

Additional versions in toolcache (`/opt/hostedtoolcache/node/`):

| Version |
|---------|
| **24.13.1** (default) |
| 22.22.0 |
| 20.20.0 |

**npm 11.8.0** is bundled with Node 24.

Global npm packages pre-installed:

| Package | Version |
|---|---|
| corepack | 0.34.6 |
| npm | 11.8.0 |

`yarn 1.22.22` is available at `/usr/local/bin/yarn`. `pnpm`, `deno`, and `bun` are **not** pre-installed.

---

## .NET

**dotnet** is in PATH. Default version: **10.0.102**.

### .NET SDKs installed

| SDK version |
|-------------|
| **10.0.102** (default) |
| 9.0.311, 9.0.205, 9.0.114 |
| 8.0.418, 8.0.319, 8.0.206, 8.0.124 |

### Runtimes installed

| Runtime | Versions |
|---------|---------|
| `Microsoft.NETCore.App` | 8.0.6, 8.0.22, 8.0.24, 9.0.6, 9.0.13, **10.0.2** |
| `Microsoft.AspNetCore.App` | same set as above |

---

## Java

**OpenJDK 17.0.18 LTS (Temurin)** is the default Java.

| Property | Value |
|---|---|
| Distribution | Eclipse Temurin (Adoptium) |
| Version | **17.0.18+8 LTS** |
| Architecture | x86_64 |
| `JAVA_HOME` | `/usr/lib/jvm/temurin-17-jdk-amd64` |
| `javac` | 17.0.18 |

Additional Temurin versions in toolcache (`/opt/hostedtoolcache/Java_Temurin-Hotspot_jdk/`):

| Version |
|---------|
| 8.0.482 |
| 11.0.30 |
| **17.0.18** (default) |
| 21.0.10 |
| 25.0.2  |

Build tools pre-installed:

| Tool | Version |
|---|---|
| Maven (`mvn`) | 3.9.12 |
| Gradle | 9.3.1 |
| Kotlin (`kotlin`) | 2.3.10 (uses JRE 17) |

---

## Go

| Property | Value |
|---|---|
| Version | **1.24.13** (default in PATH) |
| Target | `linux/amd64` |
| GOROOT | `/opt/hostedtoolcache/go/1.24.13/x64/` |

Additional versions in toolcache: 1.22.12, 1.23.12, 1.25.7.

---

## Rust

| Property | Value |
|---|---|
| rustc | **1.93.1** stable |
| cargo | 1.93.1 |
| Default host | `x86_64-unknown-linux-gnu` |

---

## Ruby

| Property | Value |
|---|---|
| Version | **3.2.3** (system, `/usr/bin/ruby`) |
| gem | 3.4.20 |

Additional versions in toolcache (`/opt/hostedtoolcache/Ruby/`): 3.2.10, 3.3.10, 3.4.8, 4.0.1.

---

## Other Languages

| Language | Version | Notes |
|---|---|---|
| PHP | 8.3.6 | `/usr/bin/php` |
| Perl | 5.38.2 | x86_64-linux-gnu-thread-multi |
| Haskell GHC | 9.14.1 | `/usr/bin/ghc` |
| Swift | 6.2.3 | `swift-6.2.3-RELEASE` |

R and Scala are **not** pre-installed.

---

## C and C++

| Tool | Version | In PATH | Notes |
|---|---|---|---|
| GCC | 13.3.0 | ✅ | Ubuntu build; `x86_64-linux-gnu` |
| Clang / LLVM | 18.1.3 | ✅ | Ubuntu clang package |
| CMake | 3.31.6 | ✅ | `/usr/local/bin/cmake` |
| Ninja | 1.13.2 | ✅ | `/usr/local/bin/ninja` |
| GNU Make | 4.3 | ✅ | `/usr/bin/make` |
| vcpkg | 2025-12-16 | ✅ | `/usr/local/share/vcpkg/vcpkg` |

---

## Version Control

| Tool | Version | Notes |
|---|---|---|
| Git | 2.53.0 | Git LFS enabled |
| GitHub CLI (`gh`) | 2.87.3 | — |
| Mercurial (`hg`) | 6.7.2 | — |

---

## Containers & Kubernetes

| Tool | Version | Notes |
|---|---|---|
| Docker | 28.0.4 | ✅ Available (unlike Windows ARM64) |
| Podman | 4.9.3 | ✅ Available |
| kubectl | 1.35.1 | Kustomize 5.7.1 |
| Helm | 3.20.0 | — |
| kind | 0.31.0 | — |
| minikube | 1.38.1 | — |

---

## Cloud & DevOps CLIs

| Tool | Version | Notes |
|---|---|---|
| AWS CLI v2 | 2.33.28 | — |
| Azure CLI (`az`) | 2.83.0 | — |
| Pulumi | v3.223.0 | — |
| Ansible | 2.20.3 (core) | via pipx |

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
| yarn | 1.22.22 | `/usr/local/bin/yarn` |

---

## Package Managers

| Manager | Version | Notes |
|---|---|---|
| pip | 24.0 | Python packages |
| npm | 11.8.0 | Node packages |
| cargo | 1.93.1 | Rust crates |
| vcpkg | 2025-12-16 | C/C++ libraries |
| yarn | 1.22.22 | JavaScript |

`apt` (Ubuntu) is available for installing additional system packages.

---

## Key Differences vs Windows ARM64

| Category | Windows ARM64 (Previous) | Ubuntu AMD64 (Current) |
|---|---|---|
| Package manager | `winget`, `choco` | `apt`, `pip`, `npm`, `cargo` |
| Docker / containers | ❌ Not available | ✅ Docker 28 + Podman 4.9 |
| Python default | 3.13.11 (ARM64 native) | 3.12.3 (x86_64) |
| Node.js | 24.13.0 | 24.13.1 |
| .NET default | 10.0.101 | 10.0.102 |
| Java default | Temurin 21.0.9 (ARM64) | Temurin 17.0.18 (x86_64) |
| Go | 1.24.11 (ARM64) | 1.24.13 (x86_64) |
| Rust | 1.92.0 (ARM64 MSVC) | 1.93.1 (x86_64 GNU) |
| GCC | MinGW-w64 14.2 (x64 emul.) | **GCC 13.3 native** |
| Clang | 20.1.6 (ARM64 native) | 18.1.3 (x86_64) |
| Swift | Not installed | **6.2.3 ✅** |
| Haskell GHC | Stack 3.7.1 | **GHC 9.14.1 ✅** |
| WSL2 | Available (no distros) | N/A |
