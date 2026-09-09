---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

FusedKernelLibrary (FKL) is a header-only C++20 library for automatic GPU kernel fusion (Vertical, Horizontal, Backwards Vertical, and Divergent Horizontal Fusion), with CPU and CUDA backends. Only nvcc is supported as the CUDA compiler. `main` is the development branch (v0.2.0, API may break); `LTS-C++17` is the frozen-API branch.

Deep per-topic docs live in `.github/skills/*/SKILL.md` (implementing operations, implementing DPPs, fusion techniques, data structures, using the library, language bindings, build/test). Read the relevant skill before nontrivial work. `.github/copilot-instructions.md` overlaps with them but contains stale claims — see "Stale documentation" below.

## Build and test commands

Requires CMake >= 3.28, a C++20 host compiler, and CUDA (nvcc). Project policy and CI treat CUDA as required, but CMake degrades gracefully: without nvcc it configures CPU-only and generates only `*_cpp` targets.

```bash
cmake -G Ninja -B build -DCMAKE_BUILD_TYPE=Release -S .   # configure
cmake --build build --config Release                       # build everything (slow: ~50 nvcc TUs)
cd build && ctest --build-config Release                   # run all tests (the merge gate)
```

Single test — each test header generates an executable target whose name equals its ctest name:

```bash
cmake --build build --target utest_softmax_cu   # build just one test
./build/bin/utest_softmax_cu                     # run directly (Ninja puts binaries in build/bin/)
ctest -R '^utest_softmax_cu$'                    # or via ctest, from build/
ctest -R '_cpp$'                                 # CPU-backend tests only (no GPU needed at runtime)
```

Key CMake options: `ENABLE_CPU` (ON), `ENABLE_CUDA` (ON if nvcc found), `BUILD_TEST` (ON), `BUILD_UTEST` (ON), `ENABLE_BENCHMARK` (OFF), `CUDA_ARCH` ("native", passed verbatim to `CUDA_ARCHITECTURES` — no arch filtering exists), `ARCH_FLAGS` (CPU SIMD), `ENABLE_NVTX`, `ENABLE_DEBUG`, `TEMPLATE_DEPTH` (1000).

There is no lint/format gate. Format manually with `clang-format -i` using the repo-root `.clang-format` (LLVM base, 4-space indent, 120-char lines, `PointerAlignment: Right`). The merge gate is the full ctest suite building and passing across CI's compiler matrix (no `-Werror` anywhere, though the skills' PR checklists ask for warning-clean nvcc and clang builds). CI runs only on PRs to `main` (self-hosted runners): Linux amd64/arm64 with g++-13 and clang++-21 + CUDA 13.3; Windows with cl 14.44 + CUDA 13.0, cl 14.51 + CUDA 13.3, and clang-cl 14.51 + CUDA 13.3.

## Test infrastructure (no framework — no GTest/Catch2)

- Tests are header files auto-discovered by CMake (`cmake/tests/discover_tests.cmake`, GLOB_RECURSE with CONFIGURE_DEPENDS) inside immediate **subdirectories** of `tests/` and `utests/`. Top-level files (`tests/main.h`, `tests/operation_test_utils.h`) are never tests. Paths containing `_common` are excluded (shared helpers). Adding a new `.h` test requires no CMake edits.
- Every test header defines `int launch()` returning 0 for pass. A generated launcher TU compiles the header twice: as C++ (`<name>_cpp` target, `ParArch::CPU`) and as CUDA (`<name>_cu`, `ParArch::GPU_NVIDIA`). The same source targets both backends because `defaultParArch` switches on `__NVCC__`.
- Backend suppression is a raw **substring** search: any occurrence of `ONLY_CU` / `ONLY_CPU` anywhere in the file (even in a comment) suppresses the `_cpp` / `_cu` target. Repo convention: `#define __ONLY_CU__`.
- Test harness helpers (`START_ADDING_TESTS`, `STOP_ADDING_TESTS`, `RUN_ALL_TESTS`, `TestCaseBuilder`, the `testCases` map) live in `tests/operation_test_utils.h`; `tests/main.h` only declares `launch()`.
- Every public alias and `build()` overload needs a utest that instantiates it — template code only breaks on instantiation (see issue #244: shipped aliases that never compiled).

## Architecture

### The Operation / IOp / DPP / Executor model

- **Operation**: a stateless static-only struct (`FK_STATIC_STRUCT` deletes all ctors) that is always **single-thread** code. Anything needing thread cooperation (shared memory, `__syncthreads`, shuffles) belongs in a **DPP** (Data Parallel Pattern), never in an Operation.
- **IOp (InstantiableOperation)** = Operation type + runtime params, produced by `Op::build(...)`. Wrappers `Read<Op>`, `ReadBack<Op>`, `Unary<Op>`, `Binary<Op>`, `Ternary<Op>`, `MidWrite<Op>`, `Write<Op>` are defined in `core/execution_model/operation_model/instantiable_operations.h`. **Types define the kernel; `build()` values are runtime parameters** — changing a value never creates a new kernel, changing a type does.
- The 11 OperationTypes and their exact `exec()` signatures are documented authoritatively in the comment table of `core/execution_model/operation_model/operation_types.h` (ReadType, WriteType, UnaryType, BinaryType, ReadBackType, TernaryType, MidWriteType, Incomplete*/Open/Closed). `IncompleteTernaryType` is declared but unused — do not implement against it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Libraries-Openly-Fused/FusedKernelLibrary](https://github.com/Libraries-Openly-Fused/FusedKernelLibrary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
