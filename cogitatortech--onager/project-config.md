---
trigger: always_on
description: This file provides guidance to coding agents collaborating on this repository.
---

# AGENTS.md

This file provides guidance to coding agents collaborating on this repository.

## Mission

Onager is a DuckDB extension that adds graph analytics functions to SQL, including centrality measures, community detection, pathfinding, graph
metrics, and graph generators.
It has two tightly coupled layers:

1. A Rust core that wraps the Graphina graph library, runs graph algorithms, validates inputs, and exposes a C ABI.
2. A C++ DuckDB extension layer that registers SQL scalar functions and table functions on top of that Rust ABI.

Priorities, in order:

1. Correctness and safety of the SQL-facing graph algorithm behavior.
2. Compatibility with supported DuckDB versions and extension CI.
3. Reliable input validation, result shape handling, and error propagation.
4. Small, well-tested changes that preserve the existing Rust/C++ boundary.

## Core Rules

- Use English for code, comments, docs, tests, and commit messages.
- Prefer focused fixes over broad refactoring.
- Preserve the existing Rust/C ABI unless the task explicitly requires changing it.
- Treat `onager/bindings/include/rust.h` as generated code. If Rust FFI signatures change, regenerate it with `make create-bindings`.
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

- `onager/src/lib.rs`: Rust crate entry point and public exports for the C ABI surface.
- `onager/src/graph.rs`: Graph data structures and conversions used across algorithms.
- `onager/src/error.rs`: Error types and last-error plumbing shared across the FFI boundary.
- `onager/src/algorithms/`: Graph algorithm implementations grouped by category (centrality, community, traversal, mst, links, metrics, generators,
  approximation, personalized, subgraphs, parallel).
- `onager/src/ffi/`: `extern "C"` functions exported to the C++ extension layer, one module per algorithm category plus `common.rs` for shared FFI
  helpers.
- `onager/bindings/onager_extension.cpp`: DuckDB extension entry point that wires up the registration functions.
- `onager/bindings/functions/`: Per-category C++ files that register and implement the SQL-facing table and scalar functions.
- `onager/bindings/include/functions.hpp`: Shared C++ helpers, version-compat macros, and registration forward declarations used by all binding files.
- `onager/bindings/include/onager_extension.hpp`: C++ extension declarations.
- `onager/bindings/include/rust.h`: Generated C header for the Rust ABI.
- `CMakeLists.txt`: Top-level CMake integration, platform detection, and Corrosion setup.
- `extension_config.cmake`: DuckDB extension wiring and linkage to the prebuilt Rust static library.
- `test/sql/`: Sqllogictest files for SQL-level extension behavior, one file per algorithm category plus registry and regression suites.
- `docs/examples/`, `docs/guide/`, `docs/reference/`: User-facing documentation served through MkDocs.
- `.github/workflows/tests.yml`: Rust tests and SQL tests in CI.
- `.github/workflows/lints.yml`: Rust formatting and clippy checks in CI.
- `.github/workflows/dist_pipeline.yml`: cross-platform extension packaging against DuckDB `main` and `v1.5.2`.
- `.github/workflows/docs.yml`: MkDocs site build.

## Architecture Notes

### Rust Core

The Rust crate owns graph-facing behavior: input validation, conversion from SQL-provided edge lists to Graphina graph types, algorithm execution,
result shaping, and error handling.
All SQL-visible behavior should ultimately reduce to deterministic Rust operations exposed through `ffi/`.

### FFI Boundary

The boundary between Rust and C++ is intentionally narrow:

- Rust returns primitive values, heap-allocated C strings, or result structs that own their own buffers.
- C++ is responsible for converting those values into DuckDB vectors and freeing Rust-allocated memory with the matching `onager_free_*` functions.
- Errors should cross the boundary through the existing last-error mechanism instead of ad hoc conventions.

When changing anything on one side of the boundary, inspect the matching code on the other side in the same change.

### DuckDB Layer

`onager/bindings/onager_extension.cpp` registers the extension and dispatches to per-category registration functions declared in `functions.hpp`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CogitatorTech/onager](https://github.com/CogitatorTech/onager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
