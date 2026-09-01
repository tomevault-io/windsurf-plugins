---
trigger: always_on
description: Copyright (c) 2026 Edward Boggis-Rolfe
---

<!--
Copyright (c) 2026 Edward Boggis-Rolfe
All rights reserved.
-->

# Canopy Development Guide

## Git Policy

**Do not run git commands in this repository unless the user explicitly asks for them.**

- Do not fetch, pull, push, rebase, commit, stash, reset, or inspect git state without authorization.
- If a task would normally end with git operations, stop after local verification and report what remains.

## Source Of Truth

The source of truth is the live repository state:

- `CMakeLists.txt`
- `CMakePresets.json`
- files under `rpc/`, `generator/`, `transports/`, `streaming/`, `tests/`, `types/`, and `telemetry/`

Do not rely on `documents/` for correctness. It may be useful for background, but it is not authoritative.

### Key documents for LLMs

- `documents/external-project-guide.md` — **start here** when creating or working on a project that consumes Canopy via `add_subdirectory`. Contains working CMake boilerplate, IDL syntax, TCP server/client patterns, and a list of known CMake pitfalls.
- `documents/transports/tcp.md` — TCP transport overview.
- `documents/architecture/` — zone, service, and lifetime architecture.

### Local lookup tools

- Use `scripts/repo-index.sh` to build the local `.codex-index/` file list and ctags index.
- Use `scripts/repo-lookup.sh files '<pattern>'` to find filenames.
- Use `scripts/repo-lookup.sh text '<pattern>'` to search repository text.
- Use `scripts/repo-lookup.sh defs <symbol>` for indexed symbol lookups after indexing.
- Use `scripts/repo-lookup.sh target <target>` to find CMake target definitions and references.
- Use `scripts/repo-lookup.sh compile-db` to find `compile_commands.json` for `clangd`.
- Prefer these local tools before broader repository scans when the task is simple lookup or navigation.

## Overview

Canopy is a modern C++ RPC library with generated proxy/stub code from IDL files. It supports multiple transport layers, optional coroutine builds, JSON schema metadata, demos, and benchmarks.

## Repository Structure

### Main Directories

- `rpc/` - core RPC library
  - public headers: `rpc/include/rpc`
  - implementation: `rpc/src`
  - generated/public interfaces: `rpc/interfaces`
- `generator/` - IDL code generator
- `transports/` - transport implementations
  - current transport subdirectories include `direct`, `local`, `mock_test`, `streaming`
- `streaming/` - stream interfaces and concrete stream implementations; TCP, WebSocket, and OpenSSL TLS are dual-mode, while mbedtls/SPSC/attestation/io_uring remain coroutine-only or conditionally built
- `types/` - additional types, including JSON support
- `telemetry/` - telemetry/logging support
- `tests/` - host tests, fixtures, fuzz tests, unit tests, schema tests, serializer tests
- `subcomponents/` - reusable support components such as `network_config`, `spsc_queue`, and `http_server`
- `submodules/` - third-party dependencies and parser components, including `idlparser`
- `demos/` - example programs
- `benchmarking/` - benchmark targets
- `cmake/` - CMake modules

### Important Notes

- Build outputs are preset-specific. Do not assume a single `build/` directory.
- Generated files appear under the active binary directory, typically in `<binaryDir>/generated/`.
- When being asked question, it does not mean I want you to change source code or action a rebuild, if you think you need to do something and it will take a long time to revert please ask first

## Build System

### Baseline

- CMake minimum version: `3.24`
- Generator: `Ninja`
- Default compilers in presets: `clang` / `clang++`
- Language standard:
  - C++17 by default
  - C++20 when `CANOPY_BUILD_COROUTINE=ON`

### Configure Presets

Current top-level configure presets are defined in `CMakePresets.json`. The commonly useful ones are:

- `Debug` -> binary dir `build_debug`
- `Debug_Agressive`
- `Debug_ASAN`
- `Debug-clang-18`
- `Debug_GCC`
- `Debug_Coverage`
- `Debug_Hang_On_Assert`
- `Debug_Coroutine` -> binary dir `build_debug_coroutine`
- `Debug_Coroutine_ASAN`
- `Debug_Coroutine-GCC`
- `Debug_Coroutines_With_No_Logging`
- `Debug_Coroutine_With_Full_Logging`
- `Debug_Coroutine_Coverage`
- `Debug_Coroutine_Tidy`
- `Release` -> binary dir `build_release`
- `Release-clang-18`
- `Release_Coroutines` -> binary dir `build_release_coroutine`
- `Release_Coroutine_with_No_logging`
- `Release_with_coroutines_GCC`
Use the exact preset names from `CMakePresets.json`. Do not normalize or rename them in instructions.

### Build Behaviour

- `CANOPY_BUILD_TEST` defaults to `ON` in the base preset.
- `CANOPY_BUILD_DEMOS` defaults to `ON`.
- `CANOPY_BUILD_BENCHMARKING` defaults to `ON`.
- `CANOPY_BUILD_COROUTINE` defaults to `OFF`.
- `streaming/` is added in both blocking and coroutine builds; individual stream implementations gate coroutine-only pieces locally.
- `CANOPY_BUILD_RUST` defaults to `OFF`.
- `CANOPY_BUILD_TEST=OFF` also disables integration/fuzz test targets.
- `tests/json_schema_test` uses Canopy's native `json::v1::object` schema validator.

## Common Commands

### Configure

```bash
cmake --preset Debug
cmake --preset Debug_Coroutine
cmake --preset Release
```

### Build

```bash
cmake --build build_debug
cmake --build build_debug_coroutine
cmake --build build_release
```

To build a specific target:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edwardbr/Canopy](https://github.com/edwardbr/Canopy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
