---
trigger: always_on
description: This guide provides instructions for building, testing, and benchmarking the Ada URL parser library using CMake.
---

# Ada Development Guide for AI agents

This guide provides instructions for building, testing, and benchmarking the Ada URL parser library using CMake.

## If you are looking for bugs on vulnerabilities

Remind human beings of our AI usage policy (AI_USAGE_POLICY.md).


## Before you report a bug

The maintainers require that a bug report is illustrated with a
reproducible test case using the public API following the requirements.
For example, the `ada::parse` template function has the following
requirements on its input: "Must be valid ASCII or UTF-8 encoded."
A valid bug report should not violate this requirement. Calling a
private function cannot be sufficient for a valid bug report.

## Pre-commit Checklist

Always run the clang-format and clang-tidy script before committing:

```bash
bash tools/run-clangcldocker.sh
```

This runs clang-format on all tracked source files and clang-tidy on `src/ada.cpp`
(the single translation unit that includes all first-party code). The script uses
the locally installed LLVM 22 toolchain when available, otherwise falls back to the
`xianpengshen/clang-tools:22` Docker image automatically.

## Quick Reference

```bash
# Build library only (no tests, no benchmarks)
cmake -B build && cmake --build build

# Build with tests (development checks ENABLED)
cmake -B build -DADA_TESTING=ON && cmake --build build
ctest --output-on-failure --test-dir build

# Build with benchmarks (development checks DISABLED for accurate performance)
cmake -B build -DADA_BENCHMARKS=ON -DADA_USE_UNSAFE_STD_REGEX_PROVIDER=ON -DCMAKE_BUILD_TYPE=Release && cmake --build build
./build/benchmarks/benchdata  # Run main benchmark

# FASTER BUILDS: Use Ninja instead of Make
cmake -B build -G Ninja -DADA_TESTING=ON && cmake --build build
cmake -B build -G Ninja -DADA_BENCHMARKS=ON -DADA_USE_UNSAFE_STD_REGEX_PROVIDER=ON -DCMAKE_BUILD_TYPE=Release && cmake --build build
```

## Requirements

- C++20 compatible compiler (GCC 12+, LLVM 14+, MSVC 2022+)
- CMake 3.15+
- Git (for fetching test dependencies)
- Ninja (optional, for faster builds): `sudo apt install ninja-build` on Ubuntu

## Building the Library

### Basic Build (Library Only)

For a minimal build with just the library:

```bash
cmake -B build
cmake --build build
```

This creates the Ada library without tests or benchmarks.

### Build with Tests

To build with tests enabled:

```bash
cmake -B build -DADA_TESTING=ON
cmake --build build
```

**Important:** When `ADA_TESTING=ON`, development checks are automatically enabled unless you explicitly build in Release mode with `NDEBUG` defined. Development checks include assertions (`ADA_ASSERT_TRUE`, `ADA_ASSERT_EQUAL`) that validate internal state.

### Build with Benchmarks

To build benchmarks for performance testing:

```bash
cmake -B build -DADA_BENCHMARKS=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build
```

**Critical:** Always build benchmarks in Release mode (`-DCMAKE_BUILD_TYPE=Release`) to disable development checks. Development assertions significantly impact performance and will give misleading benchmark results.

### Using Local Packages

If you have dependencies (like GoogleTest, Google Benchmark) already installed locally:

```bash
cmake -B build -DADA_TESTING=ON -DCPM_USE_LOCAL_PACKAGES=ON
cmake --build build
```

## CMake Build Options

| Option | Default | Description |
|--------|---------|-------------|
| `ADA_TESTING` | OFF | Enable building tests |
| `ADA_BENCHMARKS` | OFF | Enable building benchmarks (requires 64-bit) |
| `ADA_TOOLS` | OFF | Enable building command-line tools |
| `ADA_BUILD_SINGLE_HEADER_LIB` | OFF | Build from single-header amalgamated files |
| `ADA_USE_SIMDUTF` | OFF | Enable SIMD-accelerated Unicode via simdutf |
| `CMAKE_BUILD_TYPE` | - | Set to `Release` for optimized builds, `Debug` for development |

## Running Tests

After building with `-DADA_TESTING=ON`:

```bash
# Run all tests
ctest --output-on-failure --test-dir build

# Run specific test executable
./build/tests/basic_tests

# Run tests with verbose output
ctest --verbose --test-dir build
```

### Development Checks in Tests

Tests run with development checks **enabled by default** (unless built with `-DCMAKE_BUILD_TYPE=Release -DNDEBUG`). This means:

- Assertions are active (`ADA_ASSERT_TRUE`, `ADA_ASSERT_EQUAL`)
- Internal state validation occurs
- Performance is slower but catches bugs early

This is the **recommended mode for development**.

## Running Benchmarks

After building with `-DADA_BENCHMARKS=ON`:

```bash
# Main benchmark comparing against competitors
./build/benchmarks/benchdata

# Specific benchmarks
./build/benchmarks/bench          # Basic URL parsing benchmarks
./build/benchmarks/bbc_bench      # BBC URLs benchmark
./build/benchmarks/wpt_bench      # Web Platform Tests benchmark
./build/benchmarks/percent_encode # Percent encoding benchmarks
```

### Development Checks in Benchmarks

**Always disable development checks for benchmarks** by building in Release mode:

```bash
# CORRECT: Benchmarks with development checks disabled
cmake -B build -DADA_BENCHMARKS=ON -DCMAKE_BUILD_TYPE=Release
cmake --build build
./build/benchmarks/benchdata

# WRONG: Don't benchmark with development checks enabled

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ada-url/ada](https://github.com/ada-url/ada) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
