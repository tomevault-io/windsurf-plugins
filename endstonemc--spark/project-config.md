---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Endstone Spark is a native statistical profiler plugin for Minecraft Bedrock Dedicated Server (BDS). It samples native execution and allocation call stacks on Windows and Linux, aggregates them into spark-compatible profiles, and uploads them to or opens them with the standard spark viewer.

The plugin must remain safe inside a long-running server process. Sampling and allocator-hook paths have stricter constraints than ordinary plugin code: they must be bounded, avoid blocking, and defer symbolization, aggregation, compression, and network I/O to safe background or export-time code.

## Build Commands

### Prerequisites

- CMake 3.29+
- Ninja
- Conan 2
- Python 3.12 or newer for release-tool and native Python runtime tests
- Windows: LLVM clang-cl 18 or newer, Visual Studio Build Tools, and the Windows SDK
- Linux: Clang 18 or newer with libc++, libc++abi, and the static `libc++.a`/
  `libc++abi.a` archives required by the default symbol-fixture test

CI currently tests Clang 20 and clang-cl 20; the documented compiler minimum remains 18.

The repository ships `.conan2/profiles/default`, which selects clang-cl on Windows and Clang/libc++ on Linux. Do not run `conan profile detect` over this file.

### Install dependencies with Conan

```shell
python -m pip install "conan>=2,<3"
conan install . --build=missing
```

### Build with generated presets

After Conan generates the presets:

```shell
cmake --preset conan-relwithdebinfo
cmake --build --preset conan-relwithdebinfo
```

### Build with an explicit build directory

```shell
cmake -S . -B build -G Ninja \
  "-DCMAKE_TOOLCHAIN_FILE=build/RelWithDebInfo/generators/conan_toolchain.cmake" \
  "-DCMAKE_BUILD_TYPE=RelWithDebInfo"
cmake --build build
```

On Windows, run the commands from an environment where clang-cl can find the MSVC toolchain and Windows SDK.

## Testing

### Complete CTest suite

Preset build:

```shell
ctest --test-dir build/RelWithDebInfo --output-on-failure
```

Explicit `build` directory:

```shell
ctest --test-dir build --output-on-failure
```

The suite includes the offline profiler self-test, shared evidence-policy tests, and platform-specific native symbol-guesser tests.

### Focused tests

```shell
./build/RelWithDebInfo/spark_selftest --seconds=1
./build/RelWithDebInfo/spark_selftest --statistics-only
./build/RelWithDebInfo/spark_selftest --allocation-only
python tests/test_release_changelog.py
```

Use the corresponding `.exe` paths on Windows. `spark_allocation_benchmark` is a benchmark tool, not a correctness test.

## Code Style

### C++

Endstone uses **clang-format** and **clang-tidy** for code quality enforcement.

**Style Guidelines:**

- Based on Microsoft style with Stroustrup braces
- Naming conventions:
  - Classes/Structs/Enums: `CamelCase`
  - Methods: `camelBack`
  - Private/protected members: `lower_case_` (trailing underscore)
  - Local variables/parameters: `lower_case`
  - Macros: `UPPER_CASE`

### Python

**Configuration:**

- Line length: 120 characters

### Comments (all languages)
- Keep comments terse and human. Default to no comment; when one is warranted, one short line.
- No multi-line explanations, rationale, design-decision narration, or parenthetical asides.
- Do not leave "LLM notes" — comments that explain why a change was made, reference the development process, or restate what the code plainly does.
- Match the comment density and verbosity of the surrounding or original code (e.g. a port stays as terse as its upstream).

## Submitting Changes

### Commit Message Guidelines

Follow conventional commits format:

- `feat:` for new features
- `fix:` for bug fixes
- `docs:` for documentation changes
- `style:` for code style changes (formatting, etc.)
- `refactor:` for code refactoring
- `test:` for adding or updating tests
- `chore:` for maintenance tasks

Example:

```
feat: improve labeling of unresolved BDS frames

Detect function extents in stripped Windows and Linux BDS executables
Append best-effort RTTI or string-based hints to unresolved frames
Display guesses alongside the original module-relative address
Leave successfully symbolicated frames and non-BDS modules untouched
```

### Platform safety

- Sampling must stay off the BDS tick hot path except for the minimum bounded capture operation.
- Linux signal-handler code must remain async-signal-safe.
- Windows thread suspension and stack walking must restore target-thread state on every recoverable exit path. `ResumeThread` is retried up to 32 times; if restoration still fails for a live target, the process is terminated before it can remain suspended.
- Allocation hooks must remain reentrancy-safe and must never block allocator threads.
- Plugin shutdown uses bounded waits and must fail closed if health, export, viewer, or native backend work does not quiesce. The bootstrap aborts before unloading when quiescence is not proven; timed-out work is not treated as safe to unload.

## Architecture

### Source Structure

- `src/plugin.cpp` - Endstone plugin lifecycle and command dispatch (thin bootstrap)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EndstoneMC/spark](https://github.com/EndstoneMC/spark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
