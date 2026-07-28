---
trigger: always_on
description: OpenVINO Model Server (OVMS) is a high-performance inference serving platform built on top of **OpenVINO** and **OpenVINO GenAI**. The codebase is primarily **C++** with **Bazel** as the build system. Supporting infrastructure uses **Makefiles**, **Dockerfiles** (Ubuntu & Red Hat), and **batch files** (Windows).
---

# Copilot Instructions for OpenVINO Model Server (OVMS)

## Project Overview

OpenVINO Model Server (OVMS) is a high-performance inference serving platform built on top of **OpenVINO** and **OpenVINO GenAI**. The codebase is primarily **C++** with **Bazel** as the build system. Supporting infrastructure uses **Makefiles**, **Dockerfiles** (Ubuntu & Red Hat), and **batch files** (Windows).

**Performance is a top priority** — both **throughput** and **latency** are critical. Code changes should be evaluated for their performance impact. Avoid unnecessary copies, allocations, and blocking operations on the hot path.

## Repository Structure

- `src/` — Main C++ source code (server, gRPC/REST handlers, model management, pipelines, mediapipe, LLM, C API)
- `src/test/` — C++ unit tests (gtest-based); this is where most developer tests live
- `src/python/` — Python custom node bindings and related code
- `demos/` — End-user demo applications
- `client/` — Client libraries (C++, Python, Go, Java)
- `docs/` — Documentation
- `third_party/` — Third-party dependency definitions for Bazel
- `Dockerfile.ubuntu` / `Dockerfile.redhat` — Multi-stage Dockerfiles for Linux builds
- `Makefile` — Orchestrates Docker-based builds and test runs
- `*.bat` files — Windows build and setup scripts

## Code Style

- C++ style is enforced via `cpplint` and `clang-format`
- Run `make style` to check formatting
- Apache 2.0 license headers are required on all source files

## Expertise Areas

1. **OpenVINO Expertise:**
   - Proficient with OpenVINO core libraries and `ov::genai` components
   - Familiar with OpenVINO performance optimization techniques
2. **C++ Proficiency:**
   - Strong C++17 skills
   - Familiar with best practices in memory management, concurrency, and template programming
3. **Serving Infrastructure:**
   - gRPC and REST API handler design
   - Model management, pipeline orchestration, and MediaPipe integration
   - C API (`libovms_shared.so`) surface and bindings
4. **Build System Awareness:**
   - Bazel build configuration and dependency management
   - Minimizing C++ build times: forward declarations, include-what-you-use, avoiding transitive header leakage
   - Understanding of Bazel targets, build flags (`--//:distro`), and multi-stage Docker builds

## Code Review Instructions for PRs

When analyzing a Pull Request, follow this protocol:

1. Follow **C++ Core Guidelines** strictly. Include references in review comments.
2. Check for **hidden performance costs**: avoid `dynamic_cast` on the hot path; suggest `static_cast` or redesign if the type is known.
3. **Avoid copies**: ensure large data structures (tensors, buffers) are passed by reference or moved, not copied.
4. **Documentation**: ensure new public APIs have docstrings in C++ headers and Python bindings; update `docs/` as needed.
5. **Test coverage**: ensure that new features or changes have corresponding tests in `src/test/`.
6. **Formatting & safety:**
   - No `using namespace std; using namespace ov;`. Prefer explicit using with specific symbols if needed, for readability.
   - No `auto` for primitive types where it obscures readability.
   - Use `const` and `constexpr` wherever possible.
7. Pass non-fundamental values by `const` reference wherever possible.
8. Prefer member initializer lists over direct assignments in constructor bodies.
9. Verify that the result of every newly introduced function is used in at least one call site (except `void` functions).
10. Use descriptive function and variable names. Avoid duplicate code — extract common functionality into reusable utilities.
11. When initial container values are known upfront, prefer initializer-list / brace-initialization over constructing an empty container and inserting.
12. Unused functions and includes are not allowed. Build times are already long — do not add unnecessary `#include` directives. Prefer forward declarations where possible and follow the include-what-you-use principle.
    - **Forward-declare in headers, include in `.cpp`**: if a header only uses pointers or references to a type, use a forward declaration (`class Foo;`) instead of `#include "foo.hpp"`. Move the full `#include` to the `.cpp` file where the type is actually used.
    - **Keep headers self-contained but minimal**: each header must compile on its own, but should not pull in transitive dependencies that callers don't need.
    - **Prefer opaque types / Pimpl**: for complex implementation details, consider the Pimpl idiom to keep implementation-only types out of the public header entirely.
    - **Never include a header solely for a typedef or enum**: forward-declare the enum (`enum class Foo;` in C++17) or relocate the typedef to a lightweight `fwd.hpp`-style header.
13. **No dangling references or temporaries bound to `const T&`**:
    - Never use `const T&` parameters with default arguments that construct temporaries (e.g. `const std::string& param = ""`). This binds a reference to a temporary — use a function overload instead, or pass by value.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openvinotoolkit/model_server](https://github.com/openvinotoolkit/model_server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
