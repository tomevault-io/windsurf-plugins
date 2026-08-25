---
trigger: always_on
description: enables deterministic tests.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Endstone PAPI is a native C++20 PlaceholderAPI framework for
[Endstone](https://github.com/EndstoneMC/endstone) and Minecraft Bedrock Dedicated
Server. It provides a bracket placeholder parser, an owner-aware expansion registry,
and a service that resolves `{identifier:params}` placeholders through expansions
supplied by C++ or Python plugins.

PAPI is a **framework**, not a placeholder provider. The core contains **no** business
placeholders—no player name, ping, coordinates, online count, time, economy,
permissions, prefix, or similar values. Every value comes from a PlaceholderExpansion
registered by a plugin.

Architecture B is frozen: the native C++ core owns the parser, registry, service,
lifecycle, ownership, relational dispatch, and error containment. Python is a consumer,
a PlaceholderExpansion provider, and the binding/package layer. Python must never
become the PlaceholderAPI core.

## Build Commands

### Prerequisites

- CMake 3.29 or newer
- Ninja
- Conan 2
- Windows: LLVM clang-cl 20, the MSVC x64 build environment, and the Windows SDK
- Linux: Clang 20 with libc++ and libc++abi
- Python 3.10+ for the package and test tooling

The repository owns `.conan2/profiles/default` and `.conanrc`. The profile selects
RelWithDebInfo, C++20, Ninja, clang-cl on Windows, and Clang with libc++ on Linux. Do
not run `conan profile detect`, because that would replace the project profile.

### Configure and build

Run Windows commands from an x64 MSVC developer environment with LLVM's `clang-cl`
on `PATH`.

```shell
python -m pip install "conan==2.30.0"
conan install . --build=missing
cmake --preset papi-dev
cmake --build --preset papi-dev
```

The main outputs are:

- Windows: `build/RelWithDebInfo/_papi.cp314-win_amd64.pyd`
- Linux: `build/RelWithDebInfo/_papi.cpython-3xx-linux.so` (suffix varies by Python)
- Test binary: `build/RelWithDebInfo/tests/papi_test.exe` (or `.bin`)

For local Endstone development, CMake accepts
`-DFETCHCONTENT_SOURCE_DIR_ENDSTONE=<path>` to use an existing Endstone checkout.
Otherwise it fetches Endstone `v0.11.8`.

## Test Commands

### Native tests

```shell
ctest --preset papi-dev --output-on-failure
```

### Python tests

```shell
python -m pytest -q
```

### Architecture boundary tests

```shell
python tests/test_architecture_boundaries.py
```

### Changelog tooling tests

```shell
python tools/test_release_changelog.py
```

### Formatting and static analysis

```shell
# C++ formatting (check only)
git ls-files '*.cpp' '*.h' | xargs clang-format --dry-run --Werror

# C++ static analysis
python tools/run_clang_tidy.py --build-dir build/RelWithDebInfo

# Python lint and format
python -m ruff check .
python -m ruff format --check .
```

### Wheel and sdist

```shell
python -m build --wheel
python -m build --sdist
```

## Code Style

### C++ (clang-format + clang-tidy)

- Follow the repository `.clang-format` and `.clang-tidy` configuration.
- The formatting style is based on Microsoft style with Stroustrup braces.
- Naming conventions:
  - Classes, structs, and enums: `CamelCase`.
  - Methods: `camelBack`.
  - Private and protected members: `lower_case_`.
  - Local variables and parameters: `lower_case`.
  - Macros: `UPPER_CASE`.
- Formatting is mechanically enforced; do not hand-format around the formatter.
- Use C++20 with extensions disabled.
- Use `[[nodiscard]]` where ignoring a returned query or result would likely be a
  caller mistake.

### Python (ruff)

- Follow the repository Ruff configuration.
- The line length is 120 characters.
- Enabled rule families include:
  - `I` — isort.
  - `F` — pyflakes.
- Use `snake_case` for methods, functions, variables, and properties unless an
  external framework API requires otherwise.
- Preserve Endstone plugin metadata conventions where applicable.

### Comments (all languages)

- Keep comments terse and human. Default to no comment.
- For internal implementation code, when a comment is genuinely warranted, prefer
  one short line describing only a non-obvious constraint.
- Do not write multi-line explanations, rationale, design-decision narration,
  historical background, or parenthetical asides in implementation code.
- Do not leave "LLM notes": no comments explaining why a change was made, referring
  to the development, audit, or review process, or restating what the code plainly
  does.
- Do not translate obvious code into prose.
- Match the comment density and verbosity of the surrounding or original code. A
  port should remain as terse as its upstream unless a local contract genuinely
  differs.
- Comments should describe stable code constraints, not the history of how the
  implementation arrived there.

### Public API documentation

Public API documentation is a deliberate exception to the implementation-comment
brevity rule.

Project-owned public APIs should have concise documentation when callers need
information that cannot be inferred safely from the signature alone.

Document caller-visible contract only, including where relevant:

- Purpose and observable semantics.
- Parameter meaning.
- `nullptr` / `None` behavior.
- Return-value and failure semantics.
- Ownership and lifetime.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EndstoneMC/papi](https://github.com/EndstoneMC/papi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
