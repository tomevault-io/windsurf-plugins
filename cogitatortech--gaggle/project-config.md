---
trigger: always_on
description: This file provides guidance to coding agents collaborating on this repository.
---

# AGENTS.md

This file provides guidance to coding agents collaborating on this repository.

## Mission

Gaggle is a DuckDB extension for accessing Kaggle datasets from SQL.
It has two tightly coupled layers:

1. A Rust core that talks to the Kaggle API, manages cache state, validates inputs, and exposes a C ABI.
2. A C++ DuckDB extension layer that registers SQL functions, table functions, and replacement-scan behavior on top of that Rust ABI.

Priorities, in order:

1. Correctness and safety of the SQL-facing behavior.
2. Compatibility with supported DuckDB versions and extension CI.
3. Reliable dataset-path parsing, cache behavior, and error propagation.
4. Small, well-tested changes that preserve the existing Rust/C++ boundary.

## Core Rules

- Use English for code, comments, docs, tests, and commit messages.
- Prefer focused fixes over broad refactoring.
- Preserve the existing Rust/C ABI unless the task explicitly requires changing it.
- Treat `gaggle/bindings/include/rust.h` as generated code. If Rust FFI signatures change, regenerate it with `make create-bindings`.
- Do not edit vendored code under `external/` unless the task is explicitly about updating or patching a vendored dependency.
- Do not add new dependencies, network behavior, or background processes unless the requirement clearly calls for them.
- Keep docs and examples aligned with user-visible SQL behavior.

## Writing Style

- Use Oxford commas in inline lists: "a, b, and c" not "a, b, c".
- Do not use em dashes. Restructure the sentence, or use a colon or semicolon instead.
- Avoid colorful adjectives and adverbs. Write "TCP proxy" not "lightweight TCP proxy", "scoring components" not "transparent scoring components".
- Use noun phrases for checklist items, not imperative verbs. Write "redundant index detection" not "detect redundant indexes".
- Headings in Markdown files must be in the title case: "Build from Source" not "Build from source". Minor words (a, an, the, and, but, or, for, in,
  on, at, to, by, of, is, are, was, were, be) stay lowercase unless they are the first word.

## Repository Layout

- `gaggle/src/lib.rs`: Rust crate entry point and public exports for the C ABI surface.
- `gaggle/src/ffi.rs`: `extern "C"` functions exported to the C++ extension layer.
- `gaggle/src/error.rs`: Error types and last-error plumbing shared across the FFI boundary.
- `gaggle/src/config.rs`: Runtime configuration and environment-driven settings.
- `gaggle/src/utils.rs`: Shared helpers.
- `gaggle/src/kaggle/`: Kaggle-specific logic, including credentials, downloads, metadata, search, and dataset-path parsing.
- `gaggle/tests/`: Rust integration, regression, security, replacement-scan, and offline/mock-based tests.
- `gaggle/bindings/gaggle_extension.cpp`: DuckDB extension implementation that maps SQL calls to the Rust ABI.
- `gaggle/bindings/include/gaggle_extension.hpp`: C++ extension declarations.
- `gaggle/bindings/include/rust.h`: Generated C header for the Rust ABI.
- `CMakeLists.txt`: Top-level CMake integration, platform detection, and Corrosion setup.
- `extension_config.cmake`: DuckDB extension wiring and linkage to the prebuilt Rust static library.
- `test/sql/`: Sqllogictest files for SQL-level extension behavior.
- `docs/examples/`: SQL examples that should remain runnable against a local build.
- `.github/workflows/tests.yml`: Rust tests and SQL tests in CI.
- `.github/workflows/lints.yml`: Rust formatting and clippy checks in CI.
- `.github/workflows/dist_pipeline.yml`: cross-platform extension packaging against DuckDB `main` and `v1.5.2`.

## Architecture Notes

### Rust Core

The Rust crate owns Kaggle-facing behavior: credentials, dataset metadata, downloads, local cache management, file resolution, JSON shaping, and error
handling.
All SQL-visible behavior should ultimately reduce to deterministic Rust operations exposed through `ffi.rs`.

### FFI Boundary

The boundary between Rust and C++ is intentionally narrow:

- Rust returns primitive values or heap-allocated C strings.
- C++ is responsible for converting those values into DuckDB vectors and freeing Rust-allocated strings with `gaggle_free`.
- Errors should cross the boundary through the existing last-error mechanism instead of ad hoc conventions.

When changing anything on one side of the boundary, inspect the matching code on the other side in the same change.

### DuckDB Layer

`gaggle/bindings/gaggle_extension.cpp` registers scalar functions such as `gaggle_search` and `gaggle_info`, plus table-style behavior such as
`gaggle_ls` and replacement scans for `kaggle:` paths.
DuckDB API compatibility matters here. If a change touches vector access, function registration, or scans, verify against the vendored DuckDB headers
in `external/duckdb`.

### Build Integration

`make release` and `make debug` build the Rust crate first, then build DuckDB plus the extension.
`extension_config.cmake` expects a prebuilt Rust static library and links it into the DuckDB extension targets.
`CMakeLists.txt` also contains platform and Rust-target selection logic used by local builds and CI distribution builds.

## Generated and Derived Files

- `gaggle/bindings/include/rust.h` is generated from the Rust crate via `cbindgen`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CogitatorTech/gaggle](https://github.com/CogitatorTech/gaggle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
