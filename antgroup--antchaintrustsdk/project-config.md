---
trigger: always_on
description: AntChainTrustSDK is a C99 IoT framework built with CMake and Kconfig. It is
---

# Repository Guidelines

## Project Overview

AntChainTrustSDK is a C99 IoT framework built with CMake and Kconfig. It is
designed as a lightweight, multi-cloud compatible trusted on-chain SDK for
resource-constrained embedded devices, acting as a trusted anchor between
physical devices and blockchain networks for tamper-evident, attributable, and
secure IoT data submission.

The SDK provides platform abstraction, component libraries, and third-party
integrations for embedded systems. Supported targets include generic Linux
(`linux_x86` and `linux_arm`), Android NDK builds, and the SIMCom A7606E-H
platform, which uses an ARM Cortex-A7, OpenWrt Linux, and musl libc.

## Project Structure & Module Organization

Public umbrella headers live in `include/`. Implementation code is under
`source/`: `core/` contains lifecycle and dispatcher code, `components/`
contains libraries such as `cloud`, `mqtt`, `tls`, `crypto`, `kv`, `log`,
`ntp`, `queue`, and `json`, and `adapter/` contains common platform
interfaces plus Linux, Android, and
SIMCom implementations. Platform defaults are in `config/`, toolchain files in
`cmake/`, examples in `examples/`, helper scripts in `tools/`, vendored
submodules in `3rdparts/`, and tests in `tests/`.

Do not restyle or refactor `3rdparts/` unless intentionally updating a
submodule.

## Build, Test, and Development Commands

Run all commands from the repository root:

```bash
./build.sh [PLATFORM] [OPTIONS]              # See --help for full options
./build.sh linux_x86 --clean-build --test    # Linux clean build + tests
ANDROID_NDK_HOME=/path/to/android-ndk ./build.sh android
./build.sh simcom_a7606e --clean-build       # A7606E cross-compile
./build.sh --skip-config                     # Rebuild without Kconfig regen
./build.sh --clean                           # Remove build/ and exit
tools/format.sh                              # Format C, shell, and CMake
tools/genconfig.sh                           # Regenerate generated config
doxygen Doxyfile                             # Generate API docs
```

The optional `PLATFORM` argument selects a build target and copies its mapped
defconfig to `.config` before building. Omitting it uses the existing `.config`.
Available platforms include `linux_x86`, `linux_arm`, `android`, and
`simcom_a7606e`.

Run one test after building:

```bash
ctest --test-dir build --output-on-failure -R <test_name>
```

Common test selections:

```bash
ctest --test-dir build --output-on-failure -R queue_unit_test
ctest --test-dir build --output-on-failure --label-regex unit
ctest --test-dir build --output-on-failure --label-regex actrust
```

## Configuration System

Configuration is Kconfig-based. The source of truth is `.config`. Running
`tools/genconfig.sh`, also invoked by `./build.sh`, generates:

- `build/config/actrust_config.h`: C `#define` directives.
- `build/config/actrust_config.cmake`: CMake `set()` variables.

Do not edit generated `build/config/actrust_config.{h,cmake}` files directly.
Change configuration through `.config`, `config/*_defconfig`, or Kconfig
tooling, then regenerate with `tools/genconfig.sh` or the build script.

Platform defconfigs live in `config/`, for example `linux_defconfig`,
`android_defconfig`, and `simcom_a7606e_defconfig`. To add a new platform,
create a defconfig there and add the platform mapping in `build.sh`.
`CMakeLists.txt` files use `if(CONFIG_...)` guards for conditional compilation.

## Architecture

The top-level umbrella header `include/actrust.h` exposes the async client API:
`actrust_set_callback`, `actrust_init`, `actrust_deinit`, `actrust_connect`,
`actrust_disconnect`, `actrust_register`, and `actrust_data_publish`.
Bootstrap configuration is `actrust_config_t`. A worked end-to-end example
lives at `examples/hello_actrust.c`.

`source/core/` implements the public API on top of a job dispatcher and service
task. Public functions in `core_api.c` enqueue `core_job_*` jobs that the
service task drains. Internal helpers use the unprefixed naming convention,
such as `core_set_state` and `core_post_job`, and live behind
`core/core_internal.h`. Do not call core internals from components.

`source/adapter/include/` defines abstract platform interfaces:

- `device.h`: hardware ID, model, revision, firmware version.
- `network.h`: TCP/UDP sockets and DNS resolution.
- `security.h`: secure storage, key management, and crypto operations.
- `storage.h`: block-level persistent storage.
- `system.h`: mutex, semaphore, task, time, and log output.

Platform implementations live under `source/adapter/platform/<platform>/`:

- `linux/`: generic Linux using POSIX APIs, file-backed storage at
  `.actrust/storage`, and secure storage at `.actrust/security`.
- `android/`: Android NDK build using POSIX APIs where available plus Android
  log output, file-backed storage at `/data/local/tmp/actrust/storage`, and
  security slots at `/data/local/tmp/actrust/security`.
- `simcom/a7606e/`: SIMCom A7606E-H using POSIX APIs plus the SIMCom SDK
  (`libsdk.a`) for IMEI, with file-backed storage at `/data/actrust/storage`
  and secure storage at `/data/actrust/security` (backed by the Sunsea TEE).

The SIMCom A7606E-H toolchain uses GCC 13.3.0, musl 1.2.5, and ARM Cortex-A7

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antgroup/AntChainTrustSDK](https://github.com/antgroup/AntChainTrustSDK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
