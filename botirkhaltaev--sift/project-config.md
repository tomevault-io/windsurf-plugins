---
trigger: always_on
description: Guidelines for AI agents working on the sift codebase.
---

# AGENTS.md

Guidelines for AI agents working on the sift codebase.

## Project Overview

Sift is an indexed code search engine written in Rust, built around **composable on-disk indexes**. It builds indexes tuned to the search workload, then uses them to narrow candidate files before running the full regex engine.

The core architecture treats code search like database query execution: multiple index configurations can coexist, each narrowing candidates independently, with the `Indexes` registry intersecting their results. Today, Sift ships a runtime-width N-gram index that defaults to trigram width. `IndexConfig` records configured/persisted index identity, `IndexStore` owns build/open/update transactions, and `Index` is the opened query-time runtime dispatch. Future index types (AST indexes, dependency graphs, vector indexes) slot into the same architecture.

## Build & Test

```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace --all-features
```

Run all three before pushing. CI enforces the same checks on Linux, macOS, and Windows.

## Layout

| Path | Role |
|------|------|
| `crates/core/` | `sift-core`: composable index registry, query planning, candidate narrowing, search engine |
| `crates/core/src/query/` | Index-agnostic query description and candidate planning |
| `crates/core/src/index/` | `IndexConfig` / `Index` dispatch, `Indexes` registry, `IndexStore`, snapshot persistence |
| `crates/core/src/index/ngram/` | N-gram index: generic implementation plus trigram specialization (first shipped index type) |
| `crates/core/src/grep/` | Grep pipeline orchestration: bridges query planner, index registry, and search engine |
| `crates/cli/` | `sift-cli`: `sift` binary (clap CLI over core) |
| `fuzz/` | `cargo-fuzz` targets (standalone package, nightly) |
| `benchsuite/` | Comparative `rg` vs `sift` benchmarks |
| `scripts/` | `bench.sh`, `fuzz.sh`, `install.sh` |
| `skills/` | Agent usage skill for searching with `sift` (`npx skills`); CLI development → `crates/cli/AGENTS.md` |
| `docs/` | Performance snapshots, compatibility matrix |

## Key Conventions

- **No `unsafe`** except in `index/mmap.rs` (documented safety invariant).
- **Strict clippy:** workspace uses `pedantic + nursery + cargo` warnings; CI uses `-D warnings`.
- Fix lints at the root cause. `#[allow]` is **never** permitted.
- Small, focused changes; follow existing patterns in the crate you touch.
- Do not commit `target/`, `.cursor/`, local `.sift/` directories.

## Branch Names

Use short, descriptive kebab-case with a type prefix:

| Prefix | Use for |
|--------|---------|
| `feat/` | New behavior, flags, or API |
| `fix/` | Bug fixes, regressions |
| `docs/` | Documentation only |
| `chore/` | Tooling, CI, refactors with no user-visible change |

## Core API Entry Points

`IndexStore::open_or_create` → `IndexStore::build(configs, build_config)` → `Indexes::open` → `SearchQuery::new` → `SearchQuery::run(SearchExecution)`. The `--indexes` flag on `sift index build` / `sift index update` selects which `IndexConfig` values to use (defaults to all). See `crates/core/README.md`.

## Architecture & Design

Prefer the best current design over backward compatibility. Do not preserve old
APIs, signatures, names, or structures by default when a cleaner architecture is
available. Preserve compatibility only when explicitly requested or when there is
a concrete persisted-data, shipped-behavior, external-consumer, or migration
requirement.

Write idiomatic Rust. Prefer strong domain types, explicit ownership, clear error
boundaries, and small composable interfaces. Redesign weak abstractions instead
of layering new behavior on top of them.

Keep APIs general and composable. Avoid helpers, method names, or signatures that
overfit one caller, one test, one branch, or one implementation detail. Name
types and functions after the domain concept they model, not the incidental
mechanism they use.

When adding request/config structs, name them after the domain decision they
represent, not the mechanical data they carry. Avoid vague bundles such as
`Context`, `State`, `Read`, or `Options` unless those are the actual domain
terms. Prefer names like `CandidateSource`, `SnapshotValidation`, and
`IndexCoverage` that tell callers how to reason about the API.

Do not expose low-level planner knobs through higher-level APIs as loose fields.
Group related inputs behind a domain type owned by the layer making the
decision, and make each field describe a stable concept rather than a temporary
implementation detail.

When behavior has distinct cases, model those cases directly with domain types.
Use enums for real alternatives, structs for coherent grouped data, and options
structs for configurable behavior. Avoid boolean flags when a named domain type
would make intent clearer.

Separate domain decisions from side effects. Prefer pure, testable logic that
returns decisions or actions, with I/O, filesystem access, spawning, logging,
locking, and channel communication kept at clear orchestration boundaries.

## Function Evolution

Do not create `*_with_*`, `*_locked`, `*_async`, `*_new`, or similarly named
parallel variants when the new function is the old function plus one extra

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [botirkhaltaev/sift](https://github.com/botirkhaltaev/sift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
