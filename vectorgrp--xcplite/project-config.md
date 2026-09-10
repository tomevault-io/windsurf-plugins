---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

XCPlite (`libxcplite`) is a C11/C++17 implementation of the ASAM XCP measurement and calibration protocol, for XCP-on-Ethernet (TCP/UDP), targeting multicore Linux/QNX/macOS microprocessors as well as FreeRTOS microcontrollers. It provides thread-safe, lock-free instrumentation macros/functions so application code can expose stack, heap, thread-local, and global variables as measurement signals and calibration parameters to XCP tools (CANape, CANoe, etc.), with A2L (ASAM description file) generation either at runtime on-target or offline from ELF/DWARF via the Rust `xcpclient` tool.

The public C API is `inc/xcplib.h` + `inc/a2l.h`; the C++ API is `inc/xcplib.hpp` + `inc/a2l.hpp` (RAII wrappers over the C core). Core protocol/runtime implementation lives in `src/`.

`libxcplite` also serves as the C foundation for the experimental [XCP-Lite Rust](https://github.com/vectorgrp/xcp-lite) crate.

## Build system

CMake-based. **Five mutually exclusive build configurations**, each with its own build directory — never mix configurations in one build dir, since compile definitions differ:

| Configuration | Build dir | Config override header | Use case |
|---|---|---|---|
| `default` | `build/` | *(none, uses `src/xcplib_cfg.h`)* | 64-bit, on-target A2L generation, filesystem, Ethernet UDP/TCP |
| `no_a2l` | `build-no_a2l/` | `src/xcplib_no_a2l_cfg.h` | No on-target A2L; generated externally from ELF via `xcpclient` |
| `ptp` | `build-ptp/` | `src/xcplib_ptp_cfg.h` | Socket hardware timestamps; Linux + PTP-capable NIC |
| `shm` | `build-shm/` | `src/xcplib_shm_cfg.h` | Shared-memory multi-application mode (`shmtool`, `xcpdaemon`) |
| `rtos` | `build-rtos/` | `src/xcplib_rtos_cfg.h` | FreeRTOS embedded targets: reduced footprint, no filesystem, 32-bit, no on-target A2L generation |

Selected via `-DXCPLITE_CONFIGURATION=<name>` (default: `default`). Within a configuration, `XCPLITE_BUILD_EXAMPLES`, `XCPLITE_BUILD_TESTS`, `XCPLITE_BUILD_TOOLS` (all default `OFF`) control which targets get built — which targets exist depends on the active configuration (see table in `docs/BUILDING.md`). `XCPLITE_BUILD_RUST_TOOLS` builds `xcpclient`/`bintool` via cargo (any configuration). `XCPLITE_BUILD_BPF_DEMO` builds `bpf_demo` (default config, Linux only, requires libbpf).

`examples/silkit_demo` and `examples/external_example` are standalone projects with their own `CMakeLists.txt` that consume an installed xcplite via `find_package(xcplite)` — they are not built from the root project.

### Common commands

`build.sh` wraps CMake: `./build.sh [build_type] [configuration] [target] [options]` (any order; `build_type` = `debug`|`release`|`relwithdebinfo`, default `debug`; `configuration` = `default`|`no_a2l`|`ptp`|`shm`|`rtos`; `target` = `lib`|`examples`|`tests`|`tools`|`rust_tools`|`all`, default `examples`; `options` = `clean` `cleanall` `install` `install=<path>` `cargo_install` `tidy`).

```bash
./build.sh                          # library + examples, default config, debug
./build.sh tests                    # build+run default-config test suite
./build.sh shm tools                # shmtool + xcpdaemon
./build.sh ptp tools                # ptptool (Linux only)
./build.sh no_a2l examples          # no_a2l_demo(_cpp)
./build.sh rtos examples            # freertos_emu_demo (Linux/macOS host simulator)
./build.sh lib tidy                 # build lib + run clang-tidy
./build.sh clean examples           # clean rebuild
./build.sh cleanall                 # wipe all build-* dirs
CC=clang CXX=clang++ ./build.sh release all   # explicit compiler selection (respected even on existing build dirs)
```

Equivalent raw CMake:

```bash
cmake -B build -S . -DCMAKE_BUILD_TYPE=Debug -DXCPLITE_BUILD_EXAMPLES=ON
cmake --build build --parallel
cmake --build build --target hello_xcp     # single target
cmake --install build                       # installs to build/install by default
```

Switching compilers requires a fresh build directory (CMake caches the compiler in `CMakeCache.txt`).

### Running tests

Tests are plain executables built under `XCPLITE_BUILD_TESTS=ON` (default config: `a2l_test`, `cal_test`, `daq_test`, `clock_test`, `queue_test`, `xcp_test`, `type_detection_test_*`; `ptp` config: `clock_test` only). Build then run directly, e.g.:

```bash
./build.sh tests
./build/a2l_test
./build/cal_test
```

`test/test.sh [clean] [example_name]` runs the example integration tests against `build/` (not the unit test binaries above): builds each example, exercises it, and diffs generated `.a2l` files against `test/fixtures/`. Omit `example_name` to run all examples; `clean` first deletes generated `.a2l`/`.bin`/`.hex` files from the workspace root.

### Windows / QNX

Windows: MSVC via `cmake -DCMAKE_BUILD_TYPE=Debug -S . -B build-msvc -DXCPLITE_BUILD_EXAMPLES=ON` (C++20, atomics emulated, transmit queue always mutex-based — see Known limitations below). QNX: requires QNX SDP installed; use `build_qnx.bat` (Windows host) or `./build.sh Debug qcc all -q=<sdp_path> -a=<x86_64|aarch64le>` (Linux host); C++ targets excluded on SDP ≤7.0 (no `std::optional`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vectorgrp/XCPlite](https://github.com/vectorgrp/XCPlite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
