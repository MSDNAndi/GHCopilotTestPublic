# Coding Agent Software Environment — Windows AMD64 (Previous)

This document is a factual inventory of software **pre-installed and available out of the box** in the previous Windows/AMD64 GitHub Copilot Coding Agent session.

> **Hardware & OS environment:** See [`environment-windows-amd64.md`](./environment-windows-amd64.md)  
> **Current (Windows/ARM64) environment:** See [`environment-windows-arm-software.md`](./environment-windows-arm-software.md)

---

## Quick-Reference — What Is Available

| Category | Tool / Runtime | Version | In PATH |
|---|---|---|---|
| Shell | PowerShell Core | 7.4.13 | ✅ |
| VCS | Git | 2.53.0.windows.1 | ✅ |
| VCS | GitHub CLI (`gh`) | 2.87.3 | ✅ |
| Python | CPython 3.12.10 | 3.12.10 | ✅ (default) |
| Python | CPython 3.14.3 | 3.14.3 | via `py -3.14` (default `py`) |
| Python | CPython 3.13.12 | 3.13.12 | via `py -3.13` |
| Python | CPython 3.11.9 | 3.11.9 | via `py -3.11` |
| Python | CPython 3.10.11 | 3.10.11 | via `py -3.10` |
| Node.js | Node.js | 24.14.0 | ✅ |
| Node.js | yarn | 1.22.22 | ✅ |
| Node.js | corepack | bundled | ✅ |
| .NET | .NET SDK | 10.0.102 (default) | ✅ |
| .NET | .NET SDK | 8.x, 9.x | ✅ (via `global.json`) |
| .NET | .NET Framework | 4.8.09032 | ✅ |
| C/C++ | Clang / LLVM | 20.1.8 | ✅ |
| C/C++ | MSVC (`cl.exe`) | 14.44 + 14.29 | ❌ (needs `vcvarsall`) |
| C/C++ | GCC / MinGW-w64 | 15.2.0 | ✅ |
| C/C++ | CMake | 3.31.6 | ✅ |
| C/C++ | Ninja | 1.13.2 | ✅ |
| C/C++ | GNU Make | 4.4.1 | ✅ |
| C/C++ | vcpkg | 2025-12-16 | ✅ |
| Java | Eclipse Temurin JDK | 17.0.18 LTS (default) | ✅ |
| Java | Eclipse Temurin JDK | 11, 21, 25 (toolcache) | via `JAVA_HOME_*` |
| Java | Maven | 3.9.12 | ✅ |
| Java | Gradle | 9.3.1 | ✅ |
| Java | sbt | 1.12.3 | ✅ |
| Java | Kotlin | 2.3.10 | ✅ |
| Go | Go | 1.24.13 | ✅ |
| Rust | rustc / cargo | 1.93.1 stable | ✅ |
| Ruby | Ruby + gem | 3.3.10 / 3.5.22 | ✅ |
| PHP | PHP | 8.5.3 | ✅ |
| Perl | Perl | 5.42.0 | ✅ |
| R | R / Rscript | 4.5.2 | ✅ |
| Haskell | Stack | 3.9.3 | ✅ |
| IDE | Visual Studio Enterprise 2022 | 17.14.37012.4 | ❌ (GUI / `vcvarsall`) |
| Package mgr | winget | 1.11.510 | ✅ |
| Package mgr | Chocolatey (`choco`) | 2.6.0 | ✅ |
| Cloud | AWS CLI v2 | 2.33.27 | ✅ |
| Cloud | AWS SAM CLI | 1.154.0 | ✅ |
| Cloud | Azure CLI (`az`) | 2.83.0 | ✅ |
| Cloud | Pulumi | — | ✅ |
| Infra | kind | 0.31.0 | ✅ |
| Infra | kubectl | 1.35.1 | ✅ |
| Utility | curl | 8.16.0 | ✅ |
| Utility | 7-Zip (`7z`) | 26.00 | ✅ |
| Utility | tar / bsdtar | 3.8.4 | ✅ |
| Utility | jq | 1.8.1 | ✅ |
| Utility | OpenSSL | 3.6.1 | ✅ |
| Utility | ImageMagick (`magick`) | 7.1.2-15 | ✅ |
| Containers | Docker Engine | 29.1.5 | ✅ |
| Linux | WSL2 | 2.6.3.0 (kernel 6.6.87) | ✅ |

> **Note:** Unlike the previous ARM64 environment, all tools run natively as **x64** — there is no ARM64 vs emulated distinction. Docker Engine is available (it was absent on ARM64).

---

## Python

Five CPython versions are pre-installed in the toolcache. The Python Launcher (`py`) is in PATH and selects among them.

**Default (`python` / `python3`):** Python **3.12.10** — x64, built with MSVC v.1943.

> The `py` launcher defaults to Python **3.14.3** (the newest installed version). Use `python` or `python3` for 3.12.10, or `py -3.12` to explicitly target 3.12.

| Version | Toolcache path |
|---|---|
| **3.12.10** (default `python`) | `C:\hostedtoolcache\windows\Python\3.12.10\x64\` |
| 3.14.3 | `C:\hostedtoolcache\windows\Python\3.14.3\x64\` |
| 3.13.12 | `C:\hostedtoolcache\windows\Python\3.13.12\x64\` |
| 3.11.9 | `C:\hostedtoolcache\windows\Python\3.11.9\x64\` |
| 3.10.11 | `C:\hostedtoolcache\windows\Python\3.10.11\x64\` |

**pip 26.0.1** is pre-installed. Pre-installed packages (Python 3.12 global):

| Package | Version |
|---|---|
| argcomplete | 3.6.3 |
| click | 8.3.1 |
| colorama | 0.4.6 |
| packaging | 26.0 |
| pip | 26.0.1 |
| pipx | 1.8.0 |
| platformdirs | 4.9.2 |
| userpath | 1.9.2 |

`python -m venv` works out of the box. `virtualenv`, `poetry`, `pipenv`, and `conda` are **not** pre-installed.

---

## Node.js

Node.js **v24.14.0** is the default. It runs as **x64 native**.

| Version | Toolcache path |
|---|---|
| **24.14.0** (default) | `C:\hostedtoolcache\windows\node\24.14.0\x64\` |
| 22.22.0 | `C:\hostedtoolcache\windows\node\22.22.0\` |
| 20.20.0 | `C:\hostedtoolcache\windows\node\20.20.0\` |

**npm 11.9.0** is bundled with Node 24. Global npm packages pre-installed (`C:\npm\prefix`):

| Package | Version |
|---|---|
| @bazel/bazelisk | v1.28.1 |
| grunt-cli | 1.5.0 |
| gulp-cli | 3.1.0 |
| lerna | 9.0.4 |
| newman | 6.2.2 |
| yarn | 1.22.22 |

`pnpm` and `deno` and `bun` are **not** pre-installed.

---

## .NET

**dotnet** is in PATH (`C:\Program Files\dotnet\dotnet.exe`). The host runs as **x64 native**.

### .NET SDKs installed

| SDK version | Notes |
|---|---|
| **10.0.102** | Default (`dotnet --version`) |
| 9.0.311, 9.0.205, 9.0.114 | .NET 9 |
| 8.0.418, 8.0.319, 8.0.206, 8.0.124 | .NET 8 (LTS) |

### Runtimes installed

| Runtime | Versions |
|---|---|
| `Microsoft.NETCore.App` | 8.0.6, 8.0.22, 8.0.24, 9.0.6, 9.0.13, **10.0.2** |
| `Microsoft.AspNetCore.App` | same set as above |
| `Microsoft.WindowsDesktop.App` | same set as above |

### .NET Framework

**.NET Framework 4.8.09032** (Full + Client profiles) is installed. Targeting packs present for 2.0 through 4.8.

### NuGet sources pre-configured

- `nuget.org` → `https://api.nuget.org/v3/index.json`
- Microsoft Visual Studio Offline Packages (`C:\Program Files (x86)\Microsoft SDKs\NuGetPackages\`)

---

## C and C++

### Clang / LLVM 20.1.8 — in PATH, x64 native

```
clang version 20.1.8
Target: x86_64-pc-windows-msvc
Location: C:\Program Files\LLVM\bin\
```

Available LLVM tools (all in PATH): `clang`, `clang++`, `clang-cl`, `lld`, `lld-link`, `llvm-ar`, `llvm-config`, and more.

### MSVC (cl.exe) — present but NOT in PATH

MSVC must be activated via `vcvarsall.bat` before use:
```
C:\Program Files\Microsoft Visual Studio\2022\Enterprise\VC\Auxiliary\Build\vcvarsall.bat
```

Two toolset versions installed:

| Toolset | Host dirs available | Target arches |
|---|---|---|
| **14.44.35207** | `Hostarm64`, `Hostx64`, `Hostx86` | arm64, arm, x64, x86 |
| 14.29.30133 | `HostX64`, `HostX86` | arm64, arm, x64, x86 |

### GCC / MinGW-w64 15.2.0 — in PATH, x64 native

```
Target: x86_64-w64-mingw32
Location: C:\mingw64\bin\
```

Produces **x64 binaries**.

### Build systems

| Tool | Version | In PATH | Location |
|---|---|---|---|
| CMake | 3.31.6 | ✅ | `C:\Program Files\CMake\bin\` |
| Ninja | 1.13.2 | ✅ | `C:\Tools\Ninja\` |
| GNU Make | 4.4.1 | ✅ | `C:\mingw64\bin\` |
| nmake | — | ❌ | via `vcvarsall.bat` only |
| MSBuild | — | ❌ | via `dotnet msbuild` or `vcvarsall.bat` |

### Windows SDK versions installed

| SDK version | Include / Bin path |
|---|---|
| **10.0.26100.0** | `C:\Program Files (x86)\Windows Kits\10\` |
| 10.0.22621.0 | same |
| 10.0.19041.0 | same |
| 10.0.10240.0 | same |

### vcpkg — in PATH

```
vcpkg 2025-12-16  (C:\vcpkg\vcpkg.exe)
```

### Visual Studio Enterprise 2022 — installed

| Property | Value |
|---|---|
| Edition | Enterprise |
| Version | 17.14.37012.4 |
| Location | `C:\Program Files\Microsoft Visual Studio\2022\Enterprise\` |

---

## Java

**OpenJDK 17.0.18 LTS (Temurin)** is the default Java, x64 native.

| Property | Value |
|---|---|
| Distribution | Eclipse Temurin (Adoptium) |
| Version | 17.0.18+8 |
| Architecture | x64 |
| `JAVA_HOME` | `C:\hostedtoolcache\windows\Java_Temurin-Hotspot_jdk\17.0.18-8\x64\` |
| `javac` | 17.0.18 |

Additional JDK versions in toolcache (accessible via `JAVA_HOME_*` env vars):

| Version | Env var | Path |
|---|---|---|
| 8.0.482 | `JAVA_HOME_8_X64` | `...\8.0.482-8\x64\` |
| 11.0.30 | `JAVA_HOME_11_X64` | `...\11.0.30-7\x64\` |
| 21.0.10 | `JAVA_HOME_21_X64` | `...\21.0.10-7.0\x64\` |
| 25.0.2 | `JAVA_HOME_25_X64` | `...\25.0.2-10.0\x64\` |

Build tools pre-installed (all in PATH):

| Tool | Version |
|---|---|
| Maven (`mvn`) | 3.9.12 |
| Gradle | 9.3.1 |
| sbt | 1.12.3 |
| Kotlin (`kotlin`) | 2.3.10 (uses JRE 17) |

---

## Go

| Property | Value |
|---|---|
| Version | **1.24.13** (default in PATH) |
| Target | `windows/amd64` — x64 native |
| GOROOT | `C:\hostedtoolcache\windows\go\1.24.13\x64\` |

Also in toolcache: 1.22.12, 1.23.12, 1.25.7.

---

## Rust

| Property | Value |
|---|---|
| rustc | **1.93.1** stable |
| cargo | 1.93.1 |
| Default host | `x86_64-pc-windows-msvc` — x64 native, MSVC ABI |
| rustup home | `C:\Users\runneradmin\.rustup` |
| Installed targets | `x86_64-pc-windows-msvc` |

---

## Ruby

| Property | Value |
|---|---|
| Version | **3.3.10** |
| Build | `[x64-mingw-ucrt]` — x64 native |
| gem | 3.5.22 |
| Location | `C:\hostedtoolcache\windows\Ruby\3.3.10\x64\` |

Also in toolcache: 3.2.10, 3.4.8, 4.0.1.

---

## Other Languages

| Language | Version | Notes |
|---|---|---|
| PHP | 8.5.3 | `C:\tools\php\`; NTS, Visual C++ 2022 x64 |
| Perl | 5.42.0 | Strawberry Perl, x64 |
| R / Rscript | 4.5.2 | `C:\Program Files (x86)\R\R-4.5.2\bin\x64\` |
| Haskell Stack | 3.9.3 | x64 |

---

## Version Control

| Tool | Version | Notes |
|---|---|---|
| Git | 2.53.0.windows.1 | Git LFS enabled; `core.autocrlf=true` |
| GitHub CLI (`gh`) | 2.87.3 | — |
| Mercurial (`hg`) | ❌ Not installed | (was available on ARM64) |

---

## Package Managers (pre-installed)

| Manager | Version | Notes |
|---|---|---|
| winget | 1.11.510 | Windows Package Manager |
| Chocolatey (`choco`) | 2.6.0 | Many packages already installed via it |
| pip | 26.0.1 | Python packages |
| npm | 11.9.0 | Node packages |
| cargo | 1.93.1 | Rust crates |
| vcpkg | 2025-12-16 | C/C++ libraries |

---

## Cloud & DevOps CLIs

| Tool | Version | Notes |
|---|---|---|
| AWS CLI v2 | 2.33.27 | x64 native |
| AWS SAM CLI | 1.154.0 | — |
| Azure CLI (`az`) | 2.83.0 | — |
| Pulumi | — | `C:\ProgramData\chocolatey\lib\pulumi\` |
| kind | 0.31.0 | Kubernetes in Docker; amd64 binary |
| kubectl | 1.35.1 | — |

---

## General Utilities

| Tool | Version | Notes |
|---|---|---|
| curl | 8.16.0 | Schannel TLS |
| 7-Zip (`7z`) | 26.00 | x64 |
| tar / bsdtar | 3.8.4 | libarchive |
| jq | 1.8.1 | JSON processor |
| OpenSSL | 3.6.1 | `C:\Program Files\OpenSSL\bin\` |
| ImageMagick (`magick`) | 7.1.2-15 Q16-HDRI | x64 |
| NSIS | — | `C:\Program Files (x86)\NSIS\` |

---

## Containers & Virtualisation

| Feature | Status |
|---|---|
| Docker Engine | ✅ **29.1.5** — available (`windows/amd64`) |
| WSL2 | ✅ Installed (kernel 6.6.87, WSLg 1.0.71) — **no Linux distros configured** |

> Docker Engine **is available** on this AMD64 runner, unlike the previous ARM64 environment where Docker was absent.

---

## Browser Automation (WebDrivers)

Pre-installed WebDrivers for Selenium / browser testing:

| Driver | Version | Location |
|---|---|---|
| ChromeDriver | 145.0.7632.117 | `C:\SeleniumWebDrivers\ChromeDriver\` |
| EdgeDriver | 145.0.3800.70 | `C:\SeleniumWebDrivers\EdgeDriver\` |
| GeckoDriver (Firefox) | 0.36.0 | `C:\SeleniumWebDrivers\GeckoDriver\` |
| IEDriverServer | 4.14.0.0 | `C:\SeleniumWebDrivers\IEDriver\` |
| Google Chrome | 145.0.7632.117 | — |
| Microsoft Edge | 145.0.3800.70 | — |
