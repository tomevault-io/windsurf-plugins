---
trigger: always_on
description: - **sjasmplus** is a cross-platform Z80 assembler with advanced features for ZX Spectrum, ZX Next, Amstrad CPC, and related platforms.
---

# Copilot Instructions for sjasmplus

## Project Overview
- **sjasmplus** is a cross-platform Z80 assembler with advanced features for ZX Spectrum, ZX Next, Amstrad CPC, and related platforms.
- Written in C++ (see `sjasm/`), with Lua scripting support (see `lua5.5/` and `sjasm/lua_sjasm.cpp`).
- Major components: assembler core (`sjasm/`), Lua integration, and extensive test suites (`cpp-src-tests/`, `tests/`, `examples/`).

## Build & Test Workflows
- **Build (Linux/macOS):**
  - Use `make -j8` from the project root.
- **Build (Windows):**
  - Use `Makefile.win` or project files (`sjasmplus.cbp`, `sjasmplus.workspace`).
- **Run tests:**
  - Unit tests: `make test`, filter set of tests by adding TEST=<subfolder_name> (subfolders in tests/), like `make test TEST=misc`
- **CI/CD:**
  - **GitHub Actions** (`.github/workflows/`): Linux GCC builds (9/11/latest), FreeBSD clang, Windows MinGW, Windows CMake MSVC, coverage reporting.
  - **GitHub Actions** (`.github/workflows/`): CodeQL security analysis builds, and reproducibility testing (see below).
  - Cirrus CI: Cirrus was primary CI platform, but it's gone since 2026-06-01, so all CI tasks were migrated to GitHub Actions. Any mentions of Cirrus should thus describe the migration or are most likely obsolete and should be removed/replaced.
- **Binary Reproducibility Testing** (see issue #235):
  - **Workflow**: `.github/workflows/reproducibility.yml` runs weekly (Friday 8 AM UTC) or on manual trigger (`workflow_dispatch`).
  - **Tools**: `reprotest` (Debian Reproducible Builds toolkit) verifies that builds with environmental variations (users, timezones, locales, etc.) produce identical binaries.
  - **Docker setup**: `ContinuousIntegration/Dockerfile.reprotest` contains the reprotest environment; also usable locally with `docker run` for testing before release.
  - **Running locally**: `docker build -f ContinuousIntegration/Dockerfile.reprotest -t reprotest-env . && docker run --rm --privileged -v $(pwd):/workspace -w /workspace reprotest-env bash -c "reprotest --vary=+all 'make clean && make -j4' sjasmplus"`

## Assembler Architecture: Three-Pass Model
Understanding the pass structure is critical for feature development:
- **Pass 1 & 2**: Dry runs that define labels, compute expression values, and validate structure. NO machine code is emitted.
- **Pass 3**: Emits actual machine code. Code size must match Pass 2 for correctness; mismatches indicate label/expression evaluation errors.
- **Device (virtual memory)**: Acts as an extended memory array. In-memory assembly enables post-fact directives (e.g., `SAVEBIN`) and iterative content manipulation (reassemble over existing memory, save variants). Without a device, output is "pipe mode" (streaming) and doesn't support these features.
- **Implication**: Some features (e.g., memory-dependent directives) require virtual memory; validate test cases with both pipe mode and device mode.

## Key Conventions & Patterns
- **Directives and pseudo-ops** are registered in `sjasm/directives.cpp` (`InsertDirectives()`), supporting custom syntax and device-specific features.
- **Device emulation** is modular; new devices can be added in `sjasm/devices.cpp` and `sjasm/devices.h`. Device is mostly infrastructure; focus on correctness of assembler core.
- **Lua scripting** is user-facing automation and macros. Development on Lua integration only needed when extending API (see `sjasm/lua_sjasm.cpp` and `lua5.5/`); most features don't touch it.
- **Testing:**
  - Unit tests use custom test runners in `cpp-src-tests/`.
  - Many usage examples double as integration tests in `examples/` and `tests/`.
- **Documentation:**
  - Main docs: `docs/documentation.html` (generated from `docs/documentation.xml`), also available online.
  - Build/install: `Makefile`, `INSTALL.md`.
  - **Architecture & design**: The "Assembling Process" chapter covers core mechanics (three-pass model, per-line substitution rules, macro expansion). Reference this when understanding how features interact across passes.
  - Long-standing documentation gaps (e.g., #37 on substitution rules, #182 on pass mechanics) are best addressed via integrated explanatory chapters, not scattered feature notes.

## Project-Specific Notes
- **Multi-platform:**
  - Code and build scripts support Linux, macOS, Windows, and CI (see `ContinuousIntegration/` and `.github/workflows/ci.yml`).
- **High test coverage, new features are almost always developed by TDD approach.**
  - most of the features are tested end-to-end via tests in `tests/`, providing both example of usage and ensuring resulting binaries and listing files
  - `tests/` tests consist of main .asm file, all related files must have identical stem name, providing listing file .lst will allow for assembling errors (as long as listing does match, ie. errors were listed), otherwise test must assemble without errors. If assembling without errors is expected, test should rather produce binary to compare end result.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [z00m128/sjasmplus](https://github.com/z00m128/sjasmplus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
