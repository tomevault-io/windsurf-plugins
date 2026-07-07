---
trigger: always_on
description: This file is the canonical instruction guide for AI coding agents working in this repository.
---

# AGENTS.md

# AI Agent Guide for Modern C++ Repositories

This file is the canonical instruction guide for AI coding agents working in this repository.

Every agent, including Claude Code, Codex, GitHub Copilot, and other coding agents, must read this file before making changes.

The goal of this repository is to maintain a modern, safe, testable, module-based C++ codebase aligned with the current direction of ISO C++.

This guide is intentionally strict. Its purpose is to prevent AI agents from producing legacy C++, fake validation reports, unsafe ownership patterns, or broad unrelated rewrites.

Primary references:

- ISO C++ Core Guidelines: https://isocpp.org/guidelines
- C++ Core Guidelines source: https://github.com/isocpp/CppCoreGuidelines
- C++ language status: https://isocpp.org/std/status
- CMake documentation: https://cmake.org/cmake/help/latest/

---

## 1. Agent Operating Principles

Agents must optimize for correctness, safety, maintainability, and verifiability.

Before editing code, the agent must:

1. Read the relevant existing files.
2. Identify the owning subsystem.
3. Understand the module boundary.
4. Make the smallest correct change.
5. Build.
6. Test.
7. Report exact results.

Agents must not:

- Guess project behavior without reading code.
- Invent successful build or test results.
- Hide failing tests.
- Replace modern C++ with legacy patterns.
- Make broad formatting-only changes.
- Modify unrelated files.
- Add new dependencies without a strong reason.
- Ignore compiler warnings introduced by the change.

---

## 2. Toolchain Policy

The primary development path assumes a recent toolchain with strong support for modules, `import std`, concepts, and current C++26 features.

Preferred primary compiler:

```text
Clang 22 or newer
```

Supported fallback compilers may be used only when the requested feature set works correctly:

```text
Clang 18.1.2+     minimum for practical module experiments
MSVC 14.36+       when module support is verified
GCC 15+           when module and standard library support are verified
```

The preferred generator is:

```text
Ninja
```

The preferred CMake version is:

```text
CMake 4.3+
```

Minimum CMake baseline for this repository family:

```cmake
cmake_minimum_required(VERSION 3.30)
```

If the active compiler, standard library, CMake version, or generator cannot support the requested modern C++ feature, the agent must fail clearly or use the repository-provided fallback option.

The agent must not silently downgrade the code style.

---

## 3. Language Standard Policy

This repository targets C++20 or newer.

Preferred standard order:

1. C++26 when available.
2. C++23 when C++26 is not available.
3. C++20 as the minimum acceptable baseline.

New code must not be written in legacy C++ style.

Required modern C++ features when appropriate:

- Modules.
- Concepts.
- Compile-time constraints.
- `constexpr`.
- `consteval` when useful.
- `constinit` for stable static initialization.
- Ranges.
- Coroutines only when they simplify asynchronous control flow.
- `std::expected` or the project equivalent for recoverable errors.
- `std::span` for non-owning contiguous views.
- `std::string_view` for read-only string parameters with clear lifetime.
- `std::optional` for optional values.
- `std::variant` for closed alternatives.
- `std::chrono` for time.
- `std::filesystem` for paths.
- RAII for ownership and cleanup.
- Strongly typed enums.

Legacy patterns are not allowed by default.

The following are discouraged by default and require either explicit user request, a low-level systems reason, or a documented compatibility boundary:

- Raw owning pointers.
- Manual `new` and `delete`.
- C-style arrays for owned storage.
- C-style casts.
- Hidden global mutable state.
- Macro-driven business logic.
- New `.h` files.
- New `.hpp` files unless required for dependency boundaries or compatibility.
- Classic `.h` / `.cpp` architecture for new internal project code.

When such a pattern is truly necessary, the agent must document why it is necessary in the code review summary.

Acceptable reasons include:

- Low-level allocator implementation.
- ABI boundary.
- C API interop.
- Operating system API boundary.
- Third-party library compatibility.
- Embedded or performance-critical memory layout.
- Placement-new or custom lifetime management in a carefully isolated type.
- Compiler or standard library workaround.
- Explicit user instruction.

Even in those cases, isolate the unsafe or low-level pattern behind a small, tested, RAII-safe abstraction.

---

## 4. File Extension Rules

The repository uses the following extension policy:

```text
.cppm   C++ module interface / declaration
.cpp    C++ module implementation or executable entry point
.hpp    Optional dependency boundary, compatibility header, or third-party adapter
.h      Not allowed for new project files unless explicitly requested or required by external tooling
```

Declaration must live in `.cppm`.

Implementation must live in `.cpp`.

If a dependency cannot be imported as a module and requires textual inclusion, isolate it behind `.hpp`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thecompez/ai-for-modern-cpp](https://github.com/thecompez/ai-for-modern-cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
