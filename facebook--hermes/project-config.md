---
trigger: always_on
description: Enables 32-bit HermesValue encoding when possible, saving memory at the cost of some overhead:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CRITICAL: Never Change the Working Directory

**NEVER use `cd` to change the current directory from the project root.** Almost all operations can be performed by passing the correct path to commands and tools. Changing directories causes confusion and errors in subsequent operations.

In the rare cases where changing directory is absolutely unavoidable, use a subshell so the directory change does not persist:
```bash
(cd other-dir; command;)
```

## Overview

Hermes is a JavaScript engine optimized for fast start-up of React Native apps. It features ahead-of-time static optimization and compact bytecode.

## Build Commands

### Default Build: ASan+Debug with -O1

Always build and test with AddressSanitizer enabled unless the user explicitly asks otherwise or there is a specific reason not to (e.g., performance benchmarking, testing release behavior). The ASan build catches memory bugs that are otherwise silent or non-deterministic.

```bash
# Configure ASan+Debug build (the default for development)
cmake -B cmake-build-asan -G Ninja -DCMAKE_BUILD_TYPE=Debug \
  -DHERMES_ENABLE_ADDRESS_SANITIZER=ON \
  -DCMAKE_CXX_FLAGS="-O1" -DCMAKE_C_FLAGS="-O1"
```

### Other Build Configurations

```bash
# Plain Debug build (no sanitizer, for debugging with full symbols)
cmake -B cmake-build-debug -G Ninja -DCMAKE_BUILD_TYPE=Debug

# Release build
cmake -B cmake-build-release -G Ninja -DCMAKE_BUILD_TYPE=Release
```

#### Common CMake Options

Pass these with `-D` when configuring, e.g., `cmake -B build -DCMAKE_BUILD_TYPE=Debug -DHERMES_ENABLE_DEBUGGER=ON`

**Build Type & Core Options:**
- `CMAKE_BUILD_TYPE` - Debug or Release (required)
- `HERMES_ENABLE_DEBUGGER` - Build with debugger support (default: OFF)
- `HERMES_FACEBOOK_BUILD` - Build Facebook internal version (default: OFF)
- `HERMES_ENABLE_CONTRIB_EXTENSIONS` - Include community-contributed extensions (default: ON)
- `HERMES_ENABLE_WERROR` - Treat warnings as errors (default: OFF)

**Sanitizers:**
- `HERMES_ENABLE_ADDRESS_SANITIZER` - Enable ASan (default: OFF)
- `HERMES_ENABLE_UNDEFINED_BEHAVIOR_SANITIZER` - Enable UBSan (default: OFF)
- `HERMES_ENABLE_THREAD_SANITIZER` - Enable TSan (default: OFF)

**GC & Memory:**
- `HERMESVM_GCKIND` - GC type: MALLOC or HADES (default: HADES)
- `HERMESVM_HEAP_HV_MODE` - Heap HermesValue encoding mode (default: HEAP_HV_64). See "Heap HermesValue Modes" section below.
- `HERMESVM_SANITIZE_HANDLES` - Move heap after every alloc to catch stale handles (default: OFF)

**Performance/Debug Tradeoffs:**
- `HERMES_SLOW_DEBUG` - Enable slow checks in Debug builds (default: ON)
- `HERMESVM_ALLOW_JIT` - JIT mode: 0 (off), 1 (auto), 2 (force on) (default: 0)

**Compilation Modes:**
- `HERMESVM_INTERNAL_JAVASCRIPT_NATIVE` - Use natively compiled internal JS instead of bytecode (default: OFF)
- `HERMES_UNICODE_LITE` - Use internal no-op unicode instead of system libraries (default: OFF)

**External Dependencies:**
- `HERMES_ALLOW_BOOST_CONTEXT` - Use Boost.Context fibers: 0 (off), 1 (auto), 2 (force on) (default: 1)
- `JSI_UNSTABLE` - Enable JSI unstable APIs (default: ON)
- `IMPORT_HOST_COMPILERS` - Import shermes/hermesc from another build for cross-compilation

### Building

```bash
# Build
cmake --build cmake-build-asan --target hermes

# Run all tests
cmake --build cmake-build-asan --target check-hermes

# Run single test
cmake-build-asan/bin/hermes path/to/test.js

# Run test262 testsuite (ONLY run it when user asks for it)
python3 utils/test_runner.py path/to/test262/test -b cmake-build-asan/bin

# Generate the preprocessed JS file from a single test262 test.
# Then you can run the <output_file> with hermes.
python3 utils/test_runner.py <path_to_single_test262_test> -b cmake-build-asan/bin -d > <output_file>
```

### Running Tests in Claude Code on macOS

Due to a sandbox limitation in Claude Code on macOS, Python's multiprocessing module cannot create semaphores, causing the lit test runner to fail. This is a bug in the Claude Code sandbox, not in Hermes. To work around this, run tests in single-process mode:

```bash
LIT_OPTS="-j1" cmake --build cmake-build-asan --target check-hermes
```

Since single-process mode is slow (~5 minutes for all tests), use the `LIT_FILTER` environment variable to run only specific tests matching a regex:

```bash
# Run only tests matching "Array" in their path
LIT_OPTS="-j1" LIT_FILTER="Array" cmake --build cmake-build-asan --target check-hermes

# Run only tests in a specific directory
LIT_OPTS="-j1" LIT_FILTER="BCGen" cmake --build cmake-build-asan --target check-hermes
```

This workaround is only needed for Claude Code on macOS. Normal users and CI systems do not need these flags.

### Building a Single File

For faster iteration when modifying a single file `dir1/dir2/file.cpp`:

```bash
# Find the target the file belongs to
find cmake-build-asan/ -name file.cpp.o

# Build just that file (example for VM files)
cmake --build cmake-build-asan --target lib/VM/CMakeFiles/hermesVMRuntime_obj.dir/file.cpp.o
```

### Inspecting C++ File Structure

```bash
# List all functions in a file sorted by line number (requires ctags)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facebook/hermes](https://github.com/facebook/hermes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
