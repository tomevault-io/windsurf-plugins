---
trigger: always_on
description: This file provides guidance to coding agents collaborating on this repository.
---

# AGENTS.md

This file provides guidance to coding agents collaborating on this repository.

## Mission

Infera is a DuckDB extension for running machine learning inference (on ONNX models) directly from SQL.
It has two tightly coupled layers:

1. A Rust core that loads and caches ONNX models, runs inference through Tract, manages engine state, and exposes a C ABI.
2. A C++ DuckDB extension layer that registers SQL functions and table functions on top of that Rust ABI.

Priorities, in order:

1. Correctness and safety of the SQL-facing inference behavior.
2. Compatibility with supported DuckDB versions and extension CI.
3. Reliable model loading, cache behavior, and error propagation.
4. Small, well-tested changes that preserve the existing Rust/C++ boundary.

## Core Rules

- Use English for code, comments, docs, tests, and commit messages.
- Prefer focused fixes over broad refactoring.
- Preserve the existing Rust/C ABI unless the task explicitly requires changing it.
- Treat `infera/bindings/include/rust.h` as generated code. If Rust FFI signatures change, regenerate it with `make create-bindings`.
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

- `infera/src/lib.rs`: Rust crate entry point and public exports for the C ABI surface.
- `infera/src/engine.rs`: Inference engine state, model registry, and execution paths.
- `infera/src/model.rs`: ONNX model loading, metadata, and representation.
- `infera/src/config.rs`: Runtime configuration and environment-driven settings.
- `infera/src/http.rs`: Remote model fetching.
- `infera/src/error.rs`: Error types and last-error plumbing shared across the FFI boundary.
- `infera/src/ffi_utils.rs`: Shared helpers for the FFI boundary.
- `infera/bindings/infera_extension.cpp`: DuckDB extension implementation that maps SQL calls to the Rust ABI.
- `infera/bindings/include/infera_extension.hpp`: C++ extension declarations.
- `infera/bindings/include/rust.h`: Generated C header for the Rust ABI.
- `CMakeLists.txt`: Top-level CMake integration, platform detection, and Corrosion setup.
- `extension_config.cmake`: DuckDB extension wiring and linkage to the prebuilt Rust static library.
- `test/sql/`: Sqllogictest files for SQL-level extension behavior.
- `test/models/`: Sample ONNX models used by SQL tests.
- `test/concurrency/`: Concurrency and stress tests.
- `docs/examples/`: SQL examples that should remain runnable against a local build.
- `.github/workflows/tests.yml`: Rust tests and SQL tests in CI.
- `.github/workflows/lints.yml`: Rust formatting and clippy checks in CI.
- `.github/workflows/dist_pipeline.yml`: cross-platform extension packaging against DuckDB `main` and `v1.5.2`.

## Architecture Notes

### Rust Core

The Rust crate owns inference-facing behavior: model loading from local paths or URLs, ONNX parsing through Tract, engine state, model caching, tensor
shaping, JSON output formatting, and error handling.
All SQL-visible behavior should ultimately reduce to deterministic Rust operations exposed through the FFI layer.

### FFI Boundary

The boundary between Rust and C++ is intentionally narrow:

- Rust returns primitive values, heap-allocated C strings, or result structs that own their own buffers.
- C++ is responsible for converting those values into DuckDB vectors and freeing Rust-allocated memory with the matching `infera_free_*` functions.
- Errors should cross the boundary through the existing last-error mechanism instead of ad hoc conventions.

When changing anything on one side of the boundary, inspect the matching code on the other side in the same change.

### DuckDB Layer

`infera/bindings/infera_extension.cpp` registers scalar functions such as `infera_predict` and `infera_predict_from_blob`, plus table-style behavior
for
listing and inspecting loaded models.
DuckDB API compatibility matters here. If a change touches vector access, function registration, or scans, verify against the vendored DuckDB headers
in `external/duckdb`.

### Build Integration

`make release` and `make debug` build the Rust crate first, then build DuckDB plus the extension.
`extension_config.cmake` expects a prebuilt Rust static library and links it into the DuckDB extension targets.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CogitatorTech/infera](https://github.com/CogitatorTech/infera) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
