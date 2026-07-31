---
trigger: always_on
description: This file is for AI coding tools and human contributors working in this
---

# sonic-cpp Agent Guide

This file is for AI coding tools and human contributors working in this
repository. It follows the common AGENTS.md shape used by public projects:
project overview, commands, architecture, style, testing, security, and
agent-specific rules. Keep it practical: prefer facts that can be verified from
source, build files, and tests.

If a subdirectory gets its own `AGENTS.md` later, treat that more specific file
as overriding this root guide for files under that subtree.

## Project Shape

`sonic-cpp` is a header-only, SIMD-accelerated C++ JSON parser/serializer.
The public API is under namespace `sonic_json` and is normally consumed through:

- `include/sonic/sonic.h`
- `sonic_json::Document`
- `sonic_json::Node`

The README says C++11 or above, but the active CMake and Bazel builds use
C++17. Treat build files and tests as the practical source of truth.

Primary source directories:

- `include/sonic/dom/`: DOM tree, document, parser, handlers, JSON pointer.
- `include/sonic/internal/`: low-level stack, SIMD helpers, arch dispatch.
- `include/sonic/internal/arch/`: x86/Arm/SVE2 implementation details.
- `include/sonic/jsonpath/`: JSONPath query and dump helpers.
- `include/sonic/experiment/`: experimental helpers such as lazy update.
- `tests/`: GoogleTest unit tests.
- `benchmark/`: benchmark binary sources.
- `fuzz/`: CMake-only fuzz target.

Important public plumbing:

- Errors: `include/sonic/error.h`
- Allocator: `include/sonic/allocator.h`
- Internal stack: `include/sonic/internal/stack.h`
- Write buffer: `include/sonic/writebuffer.h`
- Parse flags: `include/sonic/dom/flags.h`

## Agent Workflow

Before changing behavior:

1. Read the relevant public header and its tests.
2. Search with `rg`; avoid broad filesystem scans.
3. Preserve existing API style unless the task explicitly asks for a breaking
   API change.
4. Add focused tests before or with behavior changes.
5. Run the smallest relevant test first, then the broader suite if the change
   touches shared parsing, DOM, allocator, SIMD, or serialization code.

When reviewing changes:

- Lead with correctness bugs, memory-safety risks, API compatibility problems,
  and missing tests.
- Pay special attention to allocation failure paths. Silent failure is usually
  not acceptable in new code.
- Do not simplify arch dispatch or SIMD code without checking both build flags
  and tests.

## Change Checklist

Use this checklist before handing work back:

1. The change is scoped to the requested behavior.
2. Public API compatibility has been considered and documented if affected.
3. Allocation failures either propagate `SonicError` or preserve the old object
   state for legacy APIs.
4. Relevant unit tests were added or updated.
5. The smallest relevant test was run.
6. Broader tests were run when shared parser, DOM, allocator, SIMD, or
   serialization behavior changed.
7. No build outputs, dependency caches, or benchmark artifacts were edited.

## Build And Test Commands

Quick command recap:

| Task | Command |
| --- | --- |
| Configure CMake build | `cmake -S . -B build` |
| Build CMake unit test | `cmake --build build --target unittest -j` |
| Run CMake unit test | `./build/tests/unittest` |
| Run Bazel unit test | `bazel run :unittest --//:sonic_arch=haswell --//:sonic_dispatch=static` |
| Run full Bazel helper | `bash scripts/unittest.sh -g --arch=haswell --dispatch=static` |
| Run benchmark with Bazel | `bazel run :benchmark --compilation_mode=opt` |

### CMake

Common local flow:

```bash
cmake -S . -B build
cmake --build build --target unittest -j
./build/tests/unittest
```

Useful CMake options:

- `BUILD_UNITTEST=ON` by default.
- `BUILD_FUZZ=OFF` by default.
- `BUILD_BENCH=OFF` by default.
- `ENABLE_SVE2_128=OFF` by default.

Sanitizers in CMake tests:

- `tests/CMakeLists.txt` enables ASAN by default through `ENABLE_ASAN=ON`.
- UBSAN can be enabled with `-DENABLE_UBSAN=ON`.

### Bazel

Bazel uses Bzlmod. `.bazelversion` pins the expected version.

Useful commands:

```bash
bazel run :unittest --//:sonic_arch=haswell --//:sonic_dispatch=static
bazel run :benchmark --compilation_mode=opt
bash scripts/unittest.sh -g --arch=haswell --dispatch=static
```

Bazel flags:

- `--//:sonic_arch={default|arm|sve2|westmere|haswell}`
- `--//:sonic_dispatch={static|dynamic}`
- `--//:sonic_sanitizer={no|gcc|clang}`

Note: `scripts/unittest.sh` accepts `--arch=aarch64` and `--arch=arm64`, then
maps them to Bazel's `arm` setting. When invoking Bazel directly, use
`--//:sonic_arch=arm`.

## Error Handling And Allocation Rules

Code in this repository often runs in parser, serializer, allocator, and SIMD
hot paths. Allocation failure handling must be explicit enough that callers can
distinguish resource failure from valid empty/null JSON values.

Preferred patterns:

- Parser/handler failures should return a concrete `SonicError`, especially
  `kErrorNoMem` for allocation failure.
- APIs that can fail should expose failure through the existing project style:
  `ParseResult`, `SonicError`, boolean success, or allocator error state.
- Legacy public APIs may need to keep source-compatible return types. In that
  case, preserve the previous object state on allocation failure whenever

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bytedance/sonic-cpp](https://github.com/bytedance/sonic-cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
