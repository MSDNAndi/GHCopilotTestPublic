# Coding Agent Software Environment — Windows ARM64 (Previous)

> **This document covers the previous Windows/ARM64 software environment.**  
> The current Windows/AMD64 software environment is documented in **[`environment-windows-amd64-software.md`](./environment-windows-amd64-software.md)**.

This document is a factual inventory of software **pre-installed and available out of the box** in the Windows/ARM64 GitHub Copilot Coding Agent session.

> **Hardware & OS environment:** See [`environment-windows-arm.md`](./environment-windows-arm.md)  
> **Previous (Linux/AMD64) environment:** See [`environment.md`](./environment.md)

---

## Quick-Reference — What Is Available

| Category | Tool / Runtime | Version | In PATH | Architecture |
|---|---|---|---|---|
| Shell | PowerShell Core | 7.4.13 | ✅ | ARM64 native |
| VCS | Git | 2.52.0.windows.1 | ✅ | — |
| VCS | GitHub CLI (`gh`) | 2.83.2 | ✅ | — |
| VCS | Mercurial (`hg`) | 6.3.1 | ✅ | — |
| Python | CPython 3.13.11 | 3.13.11 | ✅ (default) | ARM64 native |
| Python | CPython 3.12.10 | 3.12.10 | via `py -3.12` | ARM64 native |
| Python | CPython 3.14.2 | 3.14.2 | via `py -3.14` | ARM64 native |
| Node.js | Node.js | 24.13.0 | ✅ | x64 (emulated) |
| Node.js | yarn | 1.22.22 | ✅ | — |
| Node.js | corepack | 0.34.5 | ✅ | — |
| .NET | .NET SDK | 10.0.101 (default) | ✅ | ARM64 native |
| .NET | .NET SDK | 6.x, 8.x, 9.x | ✅ (via `global.json`) | ARM64 native |
| .NET | .NET Framework | 4.8.09221 | ✅ | — |
| C/C++ | Clang / LLVM | 20.1.6 | ✅ | ARM64 native |
| C/C++ | MSVC (`cl.exe`) | 14.44 + 14.29 | ❌ (needs `vcvarsall`) | ARM64 + x64 |
| C/C++ | GCC / MinGW-w64 | 14.2.0 | ✅ | x64 (emulated) |
| C/C++ | CMake | 4.2.1 | ✅ | — |
| C/C++ | Ninja | 1.13.2 | ✅ | — |
| C/C++ | GNU Make | 4.4.1 | ✅ | — |
| C/C++ | vcpkg | 2025-12-05 | ✅ | — |
| Java | Eclipse Temurin JDK | 21.0.9 LTS | ✅ | ARM64 native |
| Java | Maven | 3.9.11 | ✅ | — |
| Java | Gradle | 9.1.0 | ✅ | — |
| Java | sbt | 1.11.7 | ✅ | — |
| Java | Kotlin | 2.3.0 | ✅ | — |
| Go | Go | 1.24.11 | ✅ | ARM64 native |
| Rust | rustc / cargo | 1.92.0 stable | ✅ | ARM64 native (MSVC ABI) |
| Ruby | Ruby + gem | 3.4.7 / 3.6.9 | ✅ | ARM64 native |
| PHP | PHP | 8.4.15 | ✅ | x64 (emulated) |
| Perl | Strawberry Perl | 5.32.1 | ✅ | x64 (emulated) |
| R | R / Rscript | 4.5.2 | ✅ | x64 (emulated) |
| Haskell | Stack | 3.7.1 | ✅ | x64 (emulated) |
| IDE | Visual Studio Enterprise 2022 | 17.14.36811.4 | ❌ (GUI / `vcvarsall`) | ARM64 host |
| Package mgr | winget | 1.11.510 | ✅ | — |
| Package mgr | Chocolatey (`choco`) | 2.6.0 | ✅ | — |
| Cloud | AWS CLI v2 | 2.32.17 | ✅ | ARM64 native |
| Cloud | AWS SAM CLI | 1.150.1 | ✅ | — |
| Cloud | Azure CLI (`az`) | 2.81.0 | ✅ | — |
| Cloud | Pulumi | — | ✅ | — |
| Infra | kind | 0.30.0 | ✅ | amd64 (emulated) |
| Utility | curl | 8.16.0 | ✅ | — |
| Utility | 7-Zip (`7z`) | 25.01 | ✅ | x64 (emulated) |
| Utility | tar / bsdtar | 3.8.2 | ✅ | — |
| Utility | jq | 1.8.1 | ✅ | — |
| Utility | unzip | 6.00 | ✅ | — |
| Utility | OpenSSL | 3.6.0 | ✅ | — |
| Utility | ImageMagick (`magick`) | 7.1.2 | ✅ | x64 (emulated) |
| Containers | Docker / Podman | — | ❌ **not available** | — |
| Linux | WSL2 | 2.6.3.0 (kernel 6.6.87) | ✅ | ARM64 host |

---

## Python

Three ARM64-native CPython versions are pre-installed in the toolcache. The Python Launcher (`py`) is in PATH and selects among them.

**Default (`python` / `python3`):** Python **3.13.11** — ARM64, built with MSVC v.1944.

| Version | Toolcache path |
|---|---|
| **3.13.11** (default) | `C:\hostedtoolcache\windows\Python\3.13.11\arm64\` |
| 3.12.10 | `C:\hostedtoolcache\windows\Python\3.12.10\arm64\` |
| 3.14.2 | `C:\hostedtoolcache\windows\Python\3.14.2\arm64\` |

**pip 25.3** is pre-installed. Pre-installed packages (Python 3.13 global):

| Package | Version |
|---|---|
| argcomplete | 3.6.3 |
| click | 8.3.1 |
| colorama | 0.4.6 |
| packaging | 25.0 |
| pip | 25.3 |
| pipx | 1.8.0 |
| platformdirs | 4.5.1 |
| userpath | 1.9.2 |

`python -m venv` works out of the box. `virtualenv`, `poetry`, `pipenv`, and `conda` are **not** pre-installed.

---

## Node.js

Node.js **v24.13.0** is the default. It runs as **x64 under emulation** (no ARM64 Node build is present).

| Version | Toolcache path |
|---|---|
| **24.13.0** (default) | `C:\hostedtoolcache\windows\node\24.13.0\x64\` |
| 24.12.0 | `C:\hostedtoolcache\windows\node\24.12.0\` |
| 22.21.1 | `C:\hostedtoolcache\windows\node\22.21.1\` |
| 20.19.6 | `C:\hostedtoolcache\windows\node\20.19.6\` |

**npm 11.6.2** is bundled with Node 24. Global npm packages pre-installed (`C:\npm\prefix`):

| Package | Version |
|---|---|
| @bazel/bazelisk | v1.26.0 |
| grunt-cli | 1.5.0 |
| gulp-cli | 3.1.0 |
| lerna | 9.0.3 |
| newman | 6.2.1 |
| yarn | 1.22.22 |

`pnpm` and `deno` and `bun` are **not** pre-installed.

> **Note:** The `npm`/`npx` wrapper scripts in the agent's own Node path (`C:\a\_temp\ghcca-node\`) may throw `MODULE_NOT_FOUND`. Use the toolcache path directly if needed: `node C:\hostedtoolcache\windows\node\24.13.0\x64\node_modules\npm\bin\npm-cli.js`.

---

## .NET

**dotnet** is in PATH (`C:\Program Files\dotnet\dotnet.exe`). The host runs as **ARM64 native** (`RID: win-arm64`).

### .NET SDKs installed

| SDK version | Notes |
|---|---|
| **10.0.101** | Default (`dotnet --version`) |
| 9.0.308, 9.0.205, 9.0.112 | .NET 9 |
| 8.0.416, 8.0.319, 8.0.206, 8.0.122 | .NET 8 (LTS) |
| 6.0.428, 6.0.321, 6.0.203, 6.0.136 | .NET 6 |

### Runtimes installed

| Runtime | Versions |
|---|---|
| `Microsoft.NETCore.App` | 6.0.5, 6.0.26, 6.0.36, 8.0.6, 8.0.22, 9.0.6, 9.0.11, **10.0.1** |
| `Microsoft.AspNetCore.App` | same set as above |
| `Microsoft.WindowsDesktop.App` | same set as above |

### .NET Framework

**.NET Framework 4.8.09221** (Full + Client profiles) is installed. Targeting packs present for 4.5.2 through 4.8.1.

### Pre-installed workloads

| Workload | Version |
|---|---|
| android | 36.0.0-rc.2.332 |
| ios | 26.0.10970-net10-rc.2 |
| maccatalyst | 26.0.10970-net10-rc.2 |
| maui-windows | 10.0.0-rc.2.25504.7 |
| wasm-tools | 10.0.101 |

### Pre-installed global .NET tools

| Tool | Version | Command |
|---|---|---|
| nbgv | 3.9.50 | `nbgv` |

### NuGet sources pre-configured

- `nuget.org` → `https://api.nuget.org/v3/index.json`
- Microsoft Visual Studio Offline Packages (`C:\Program Files (x86)\Microsoft SDKs\NuGetPackages\`)

---

## C and C++

### Clang / LLVM 20.1.6 — in PATH, ARM64 native

```
clang version 20.1.6
Target: aarch64-pc-windows-msvc
Location: C:\Program Files\LLVM\bin\
```

Available LLVM tools (all in PATH): `clang`, `clang++`, `clang-cl`, `lld`, `lld-link`, `llvm-ar`, `llvm-config`, and more.

### MSVC (cl.exe) — present but NOT in PATH

MSVC must be activated via `vcvarsall.bat` before use:
```
C:\Program Files\Microsoft Visual Studio\2022\Enterprise\VC\Auxiliary\Build\vcvarsall.bat
```

Two toolset versions installed, each supporting multiple host/target combinations:

| Toolset | Host dirs available | Target arches |
|---|---|---|
| **14.44.35207** | `Hostarm64`, `Hostx64`, `Hostx86` | arm64, arm, x64, x86 |
| 14.29.30133 | `HostX64`, `HostX86` | arm64, arm, x64, x86 |

For ARM64-native compilation, use: `Hostarm64\arm64\cl.exe` from toolset 14.44.

### GCC / MinGW-w64 14.2.0 — in PATH, x64 (emulated)

```
Target: x86_64-w64-mingw32
Locations: C:\mingw64\bin\ and C:\Strawberry\c\bin\
```

Produces **x64 binaries** (not ARM64). Runs under x64 emulation on this ARM64 host.

### Build systems

| Tool | Version | In PATH | Location |
|---|---|---|---|
| CMake | 4.2.1 | ✅ | `C:\Program Files\CMake\bin\` |
| Ninja | 1.13.2 | ✅ | `C:\Tools\Ninja\` |
| GNU Make | 4.4.1 | ✅ | `C:\mingw64\bin\` |
| nmake | — | ❌ | via `vcvarsall.bat` only |
| MSBuild 18.0.6 | — | ❌ | via `dotnet msbuild` or `vcvarsall.bat` |

### Windows SDK versions installed

| SDK version | Include / Bin path |
|---|---|
| **10.0.26100.0** | `C:\Program Files (x86)\Windows Kits\10\` |
| 10.0.22621.0 | same |
| 10.0.19041.0 | same |
| 10.0.10240.0 | same |

### vcpkg — in PATH

```
vcpkg 2025-12-05  (C:\vcpkg\vcpkg.exe)
```

### Visual Studio Enterprise 2022 — installed

| Property | Value |
|---|---|
| Edition | Enterprise |
| Version | 17.14.36811.4 |
| Location | `C:\Program Files\Microsoft Visual Studio\2022\Enterprise\` |

---

## Java

**OpenJDK 21.0.9 LTS (Temurin)** is the default Java, ARM64 native.

| Property | Value |
|---|---|
| Distribution | Eclipse Temurin (Adoptium) |
| Version | 21.0.9+10-LTS |
| Architecture | ARM64 (aarch64) |
| `JAVA_HOME` | `C:\hostedtoolcache\windows\Java_Temurin-Hotspot_jdk\21.0.9-10.0\aarch64\` |
| `javac` | 21.0.9 |

Also in toolcache: Temurin 23.0.2.

Build tools pre-installed (all in PATH):

| Tool | Version |
|---|---|
| Maven (`mvn`) | 3.9.11 |
| Gradle | 9.1.0 |
| sbt | 1.11.7 |
| Kotlin (`kotlin`) | 2.3.0 (uses JRE 21) |

---

## Go

| Property | Value |
|---|---|
| Version | **1.24.11** (default in PATH) |
| Target | `windows/arm64` — ARM64 native |
| GOROOT | `C:\hostedtoolcache\windows\go\1.24.11\arm64\` |

Also in toolcache: 1.22.12, 1.23.12, 1.25.5.

---

## Rust

| Property | Value |
|---|---|
| rustc | **1.92.0** stable |
| cargo | 1.92.0 |
| Default host | `aarch64-pc-windows-msvc` — ARM64 native, MSVC ABI |
| rustup home | `C:\Users\runneradmin\.rustup` |
| Installed targets | `aarch64-pc-windows-msvc` |

---

## Ruby

| Property | Value |
|---|---|
| Version | **3.4.7** |
| Build | `+PRISM [aarch64-mingw-ucrt]` — ARM64 native |
| gem | 3.6.9 |
| Location | `C:\hostedtoolcache\windows\Ruby\3.4.7\aarch64\` |

---

## Other Languages

| Language | Version | Architecture | Notes |
|---|---|---|---|
| PHP | 8.4.15 | x64 emulated | `C:\tools\php\`; NTS, Visual C++ 2022 |
| Perl (Strawberry) | 5.32.1 | x64 emulated | `C:\Strawberry\perl\bin\` |
| R / Rscript | 4.5.2 | x64 emulated | `C:\Program Files (x86)\R\R-4.5.2\bin\x64\` |
| Haskell Stack | 3.7.1 | x64 emulated | `C:\hostedtoolcache\windows\stack\3.7.1\x64\` |

---

## Version Control

| Tool | Version | Notes |
|---|---|---|
| Git | 2.52.0.windows.1 | Git LFS enabled; `core.autocrlf=true` |
| GitHub CLI (`gh`) | 2.83.2 | — |
| Mercurial (`hg`) | 6.3.1 | — |

---

## Package Managers (pre-installed)

| Manager | Version | Notes |
|---|---|---|
| winget | 1.11.510 | Windows Package Manager |
| Chocolatey (`choco`) | 2.6.0 | Many packages already installed via it |
| pip | 25.3 | Python packages |
| npm | 11.6.2 | Node packages |
| cargo | 1.92.0 | Rust crates |
| vcpkg | 2025-12-05 | C/C++ libraries |

---

## Cloud & DevOps CLIs

| Tool | Version | Notes |
|---|---|---|
| AWS CLI v2 | 2.32.17 | ARM64 native |
| AWS SAM CLI | 1.150.1 | — |
| Azure CLI (`az`) | 2.81.0 | — |
| Pulumi | — | `C:\ProgramData\chocolatey\lib\pulumi\` |
| kind | 0.30.0 | Kubernetes in Docker; amd64 binary |
| Aliyun CLI | — | `C:\aliyun-cli\` |

---

## General Utilities

| Tool | Version | Notes |
|---|---|---|
| curl | 8.16.0 | Schannel TLS |
| 7-Zip (`7z`) | 25.01 | x64 emulated |
| tar / bsdtar | 3.8.2 | libarchive |
| jq | 1.8.1 | JSON processor |
| unzip | 6.00 | Info-ZIP |
| OpenSSL | 3.6.0 | `C:\Program Files\OpenSSL\bin\` |
| ImageMagick (`magick`) | 7.1.2-10 Q16-HDRI | x64 emulated |
| NSIS | — | `C:\Program Files (x86)\NSIS\` |

---

## Containers & Virtualisation

| Feature | Status |
|---|---|
| Docker Desktop | ❌ Not installed |
| Podman | ❌ Not installed |
| WSL2 | ✅ Installed (kernel 6.6.87, WSLg 1.0.71) — **no Linux distros configured** |

---

## Browser Automation (WebDrivers)

Pre-installed WebDrivers for Selenium / browser testing:

| Driver | Location |
|---|---|
| ChromeDriver | `C:\SeleniumWebDrivers\ChromeDriver\` |
| EdgeDriver | `C:\SeleniumWebDrivers\EdgeDriver\` |
| GeckoDriver (Firefox) | `C:\SeleniumWebDrivers\GeckoDriver\` |
| Google Chrome | v143 |

---

## ARM64-Native vs x64-Emulated Summary

| Runtime / Tool | Execution |
|---|---|
| Python 3.13 / 3.12 / 3.14 | ✅ ARM64 native |
| .NET (all versions) | ✅ ARM64 native |
| Go 1.24 | ✅ ARM64 native |
| Rust 1.92 | ✅ ARM64 native (MSVC ABI) |
| Java 21 (Temurin) | ✅ ARM64 native |
| Ruby 3.4 | ✅ ARM64 native |
| Clang 20 | ✅ ARM64 native (produces ARM64 binaries) |
| MSVC 14.44 (`Hostarm64`) | ✅ ARM64 native (produces ARM64 binaries) |
| Node.js 24 | ⚠️ x64 emulated |
| GCC/MinGW 14.2 | ⚠️ x64 emulated (produces x64 binaries) |
| PHP 8.4 | ⚠️ x64 emulated |
| Perl 5.32 | ⚠️ x64 emulated |
| R 4.5 | ⚠️ x64 emulated |
| Haskell Stack 3.7 | ⚠️ x64 emulated |
| ImageMagick 7.1 | ⚠️ x64 emulated |
| MSVC 14.29 (`HostX64`) | ⚠️ x64 emulated (can cross-compile to ARM64) |
