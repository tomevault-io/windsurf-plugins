---
trigger: always_on
description: XDP for Windows is a high-performance Windows packet processing framework inspired by Linux XDP. It enables sending and receiving network packets at high rates by bypassing most of the OS networking stack.
---

# Copilot Instructions for XDP for Windows

## Repository Overview

XDP for Windows is a high-performance Windows packet processing framework inspired by Linux XDP. It enables sending and receiving network packets at high rates by bypassing most of the OS networking stack.

**Key facts:**
- **Languages**: C (kernel drivers), C++ (user-mode tests/samples), PowerShell (build/test scripts)
- **Frameworks**: Windows Driver Kit (WDK), NDIS, TAEF (test framework)
- **Target**: Windows Server 2019/2022 and Prerelease builds, x64 and arm64 platforms
- **Build Tool**: MSBuild via Visual Studio Developer Command Prompt

## Build Instructions

### Prerequisites
- Visual Studio 2022+ with:
  - "Desktop development with C++" workload
  - Latest Spectre-mitigated libs
  - C++ Address Sanitizer
  - C++ Clang Compiler for Windows
- NuGet CLI 6.3.1+
- Git submodules initialized (`git submodule update --init --recursive`)

### Building

**Always run builds from a Visual Studio Developer Command Prompt.**

1. Prepare the machine (downloads dependencies):
   ```powershell
   .\tools\prepare-machine.ps1 -ForBuild
   ```

2. Restore NuGet packages (required before every build if packages changed):
   ```powershell
   msbuild.exe xdp.sln /t:restore /p:RestoreConfigFile=tools/nuget.config /p:Configuration=Debug /p:Platform=x64
   ```

3. Prepare for eBPF compilation (if building eBPF programs):
   ```powershell
   .\tools\prepare-machine.ps1 -ForEbpfBuild
   ```

4. Build the solution:
   ```powershell
   msbuild xdp.sln /m /p:Configuration=Debug /p:Platform=x64 /p:SignMode=TestSign /p:IsAdmin=true /nodeReuse:false
   ```

**Or use the convenience script:**
```powershell
.\tools\build.ps1
```

**Important**: Use `/nodeReuse:false` to avoid cached state issues with WDK's build tasks.

### Build Configurations
- **Config**: `Debug` or `Release`
- **Platform**: `x64` or `arm64`
- **Build artifacts**: `artifacts\bin\{Platform}_{Config}\`

## Testing

All testing must occur on a separate test machine. Do not try to run
tests locally. **Always pass `-ComputerName <machine>` explicitly on
every test command** — see the `xdp-testing` skill below.

For everything else about test execution and bugcheck recovery
(remote PowerShell setup, kd attach over named pipe, the
inject/build/bugcheck/`.reboot`/`check-drivers -Force` recovery loop,
how to derive default test parameters from CI, the persist-until-clean
recovery doctrine, opt-in `-Force` recovery flags), use the
[`xdp-testing`](./skills/xdp-testing/SKILL.md) skill. The skill is
auto-loaded whenever an agent runs any `tools\*.ps1` test or recovery
command, attaches kd, or analyzes a bugcheck.

Quick reference for the most common scripts:

```powershell
.\tools\functional.ps1 -ComputerName <machine> -ListTestCases
.\tools\functional.ps1 -ComputerName <machine> -TestCaseFilter "Name=GenericBinding"
.\tools\spinxsk.ps1   -ComputerName <machine> -Minutes 5
.\tools\check-drivers.ps1 -ComputerName <machine> -Force
```

## Project Layout

```
xdp.sln                 # Main solution file
src/                    # Source code
  xdp/                  # Core XDP driver (xdp.sys)
  xdplwf/               # LWF (Lightweight Filter) driver
  xdpapi/               # User-mode API library (deprecated, use header-only API)
  xdpetw/               # ETW manifest
  xdpinstaller/         # MSI installer
  xdpruntime/           # Runtime packaging
published/external/     # Public API headers
  afxdp.h               # AF_XDP socket API
  afxdp_experimental.h  # Experimental AF_XDP features
  xdpapi.h              # XDP control API
  xdp/                  # XDP kernel/user shared definitions
test/                   # Test code
  functional/           # TAEF functional tests
    lib/tests.cpp       # Test implementation
    taef/tests.cpp      # TAEF wrappers
  spinxsk/              # Stress test tool
  xdpmp/                # Test miniport driver
samples/                # Sample applications
  rxfilter/             # RX filtering sample
  xskfwd/               # XSK forwarding sample
tools/                  # PowerShell scripts
  build.ps1             # Build script
  functional.ps1        # Functional test runner
  prepare-machine.ps1   # Machine setup
  setup.ps1             # Component installation
submodules/             # Git submodules
  cxplat/               # Cross-platform utilities
  ndis-driver-library/  # NDIS helpers
  wil/                  # Windows Implementation Libraries
```

## CI Pipeline

GitHub Actions CI runs on pull requests (`.github/workflows/ci.yml`):
1. **Build**: Debug/Release × x64/arm64
2. **Functional Tests**: Windows 2022/Prerelease
3. **Stress Tests (spinxsk)**: With driver verifier and fault injection
4. **Performance Tests**: XSK perf, ring perf, RX filter perf
5. **Fuzz Tests**: Packet parsing fuzzer (pktfuzz)
6. **CodeQL**: Security analysis (on scheduled runs)

CI uses test-signed drivers and requires test signing enabled on test machines.

## Adding Tests

When adding functional tests:
1. Add test implementation to `test/functional/lib/tests.cpp`
2. Add function declaration to `test/functional/lib/tests.h`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/xdp-for-windows](https://github.com/microsoft/xdp-for-windows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
