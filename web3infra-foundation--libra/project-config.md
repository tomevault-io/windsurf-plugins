---
trigger: always_on
description: This repository (“libra”) implements the core Git engine re-written and extended in Rust: object storage, commit-graph, packfile reader/writer, MIDX/multi-pack index, worktree semantics and transport layer. It’s foundational to the larger monorepo ecosystem at Web3Infra Foundation and is designed for large-scale, multi-client, content-addressed version control.
---

# Libra – Repository Custom Instructions for GitHub Copilot

## What this repo is

This repository (“libra”) implements the core Git engine re-written and extended in Rust: object storage, commit-graph, packfile reader/writer, MIDX/multi-pack index, worktree semantics and transport layer. It’s foundational to the larger monorepo ecosystem at Web3Infra Foundation and is designed for large-scale, multi-client, content-addressed version control.
When generating code or design suggestions, assume this context: high concurrency, large object graphs, cross-crate modular architecture, Git compatibility (SHA-1 & SHA-256), performance/memory sensitivity, and Rust ecosystem conventions.

## Languages & defaults

- Primary language: Rust (edition 2021 or later).
- Async runtime: Tokio. Logging/tracing: tracing crate.
- Error handling: for libraries use thiserror; for binaries/tests/tools use anyhow.
- Serialization: serde; CLI argument parsing: clap.
- Use unsafe only when absolutely necessary — if used, include // SAFETY: comment with invariants and add tests.
- Minor scripting / tooling around the engine may use Python or Bash, but core logic should stay in Rust.

## Build & run

- Local iterative workflow: cargo build -p <crate>, cargo test -p <crate>, cargo bench -p <crate>.

## Repository architecture & major components

- Crate layout:
    - engine/ — core Git object engine (loose objects, object lookup, packfile read/write).
    - delta/ — delta-chain rewrite engines, multi-pack index support.
    - transport/ — network layer for fetch/push over Git protocol.
    - storage/ — content-addressed storage abstraction, pack caches, object caches.
    - cli/ — command-line utilities, interactive inspection, diagnostics.
    - common/ — shared utilities (hashing, fan-out tables, error types).
- Avoid hard-coding paths or assumptions about repository size; design for millions of objects, multiple packfiles, multi-client concurrency.

## Coding style & quality

- Enforce rustfmt defaults. New/changed code should compile with zero warnings under cargo build --all-targets.
- Treat clippy warnings as errors on new code (e.g., #![deny(clippy::all)] in new crates).
- **Avoid `unwrap()` / `expect()`** in production code (library, CLI commands, internal modules, including startup/initialization). Prefer returning `Result` and propagating errors with `?`, attaching user-friendly context via `anyhow::Context` (`.context("...")` / `.with_context(|| format!(...))`) or domain-specific `thiserror` variants. `unwrap()`/`expect()` are acceptable only in **unit/integration tests** and where the logic is **obviously infallible** (e.g., compile-time-known constants) with a brief `// INVARIANT:` comment. All other code must use graceful error propagation with actionable, user-friendly messages. When reviewing code, flag `unwrap()` / `expect()` in other contexts and suggest a `Result`-based alternative.
- All errors surfaced to the user must be human-readable and actionable. Avoid exposing raw internal errors; wrap them with context that explains what went wrong, which resource was affected, and how to fix it.
- Prefer iterator/slice APIs over heap allocations in hot paths. Use SmallVec, bytes, or no-std-friendly patterns when relevant.
- Document performance expectations for critical code paths (e.g., “expected throughput > X objects/second”, “allocation count < Y per object”).

## Performance & memory

- This engine targets very large repositories — focus on streaming I/O, minimal copying, O(n) algorithms, bounded memory overhead.
- When dealing with packfiles: consider fan-out tables, delta-chain depth, object reuse, object relocation, compression with zstd or deflate.
- Support both SHA-1 and SHA-256 object IDs; avoid assumptions about 20-byte vs 32-byte lengths.
- Provide micro-benchmarks (via criterion) for hot paths; include allocation and throughput metrics. If a change causes a regression (e.g., >5% drop in throughput or >10% increase in allocation count), document it in the PR description and update benchmark results files. Significant regressions (>10% performance drop or allocation increase) should also be noted in CHANGELOG.md.

## Git compatibility & hashing

- Must interoperate with standard Git objects, refs, packfile formats, index formats.
- Support both legacy SHA-1 and new SHA-256 object IDs; design migration paths and dual-stack invariants.
- When generating code proposals: explicitly document trade-offs (compatibility vs performance).
- Avoid assumptions like “object ID is 20 bytes” or “fan-out table always 256 entries” unless clearly parameterized.

## API & CLI guidelines

- Public crates: define stable, versioned APIs; avoid leaking internal pack/graph structures unless explicitly marked unstable.
- CLI tools: default to safe, read-only operations. Provide --dry-run, --json output where appropriate.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [web3infra-foundation/libra](https://github.com/web3infra-foundation/libra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
