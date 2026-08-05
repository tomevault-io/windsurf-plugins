---
trigger: always_on
description: This file provides guidance to coding agents collaborating on this repository.
---

# AGENTS.md

This file provides guidance to coding agents collaborating on this repository.

## Mission

Graphina is a graph data science library for Rust.
It provides graph data structures and a wide range of ready-to-use algorithms for analyzing real-world networks, such as social, transportation, and
biological networks.
The goal is to be as feature-rich as NetworkX while keeping the speed and performance of Rust, and to offer a higher-level API than libraries like
petgraph and rustworkx.
A companion Python library, PyGraphina, exposes Graphina to Python through maturin.
Priorities, in order:

1. Correct, well-tested implementations of graph algorithms.
2. Clean, idiomatic Rust with safe abstractions and a high-level, ergonomic API.
3. Clear separation between the core library and the optional, feature-gated extensions.
4. Maintainable code with consistent error handling and documentation.

## Core Rules

- Use English for code, comments, docs, and tests.
- Never use `.unwrap()` or `.expect()` in non-test code (enforced by `make lint` via `clippy::unwrap_used` and `clippy::expect_used`). Production code
  should never panic.
- Algorithms return `Result<_, graphina::core::error::GraphinaError>`. Selector-style helpers that pick nodes (like `voterank`) may return plain
  collections.
- Top-level extension modules may depend only on `core`, never on each other (enforced by `make check-module-deps`).
- Gate every extension behind its feature flag with `#[cfg(feature = "...")]`. Enable only the required features to minimize size and compile time.
- Prefer small, focused changes over large refactoring.
- Add comments only when they clarify non-obvious behavior.
- Do not add features, error handling, or abstractions beyond what is needed for the current task.
- Add tests for every bug fix and new feature to prevent regression.
- Follow red-green TDD: write a failing test first, then the code to pass it (see Test-Driven Development).

Quick examples:

- Good: add a `local_reaching_centrality` function in `src/centrality/other.rs` that returns `Result<NodeMap<f64>, GraphinaError>`, gated behind
  `#[cfg(feature = "centrality")]`, with unit tests for the empty-graph and disconnected-graph cases.
- Good: add a property-based invariant to `tests/property_based_tests.rs` asserting that `connected_components` partitions every node into exactly one
  component.
- Bad: write `use crate::metrics::triangles;` inside `src/community/` to reuse a triangle count. Extensions may import only from `core`; move the
  shared helper into `core` or duplicate the small piece.
- Bad: call `.unwrap()` on a `Result` in non-test code because "the graph is obviously non-empty". Production code must never panic; return a
  `GraphinaError` instead.
- Bad: add a new algorithm without a feature gate, so it compiles into the `default = []` build.

## Writing Style

- Use Oxford commas in inline lists: "a, b, and c" not "a, b, c".
- Do not use em dashes. Restructure the sentence, or use a colon or semicolon instead.
- Avoid colorful adjectives and adverbs. Write "graph generator" not "powerful graph generator".
- Prefer using noun phrases for checklist items, not imperative verbs. Write "negative weight detection" not "detect negative weights".
- Headings in Markdown files must be in title case: "Build from Source" not "Build from source". Minor words
  (a, an, the, and, but, or, for, in, on, at, to, by, of, with, from) stay lowercase unless they are the first word.
- Write correct and complete sentences.
- Avoid made-up words, abbreviations, and colons in the middle of sentences.
- Don't use pretentious language and made-up words.

## Repository Layout

- `src/core/`: Always-enabled core library. Basic graph types, builders, IO, serialization, shortest paths, validation, and generators.
- `src/centrality/`, `src/community/`, `src/links/`, `src/metrics/`, `src/mst/`, `src/traversal/`, `src/approximation/`, `src/parallel/`,
  `src/subgraphs/`: Optional extensions, each behind a Cargo feature of the same name. The `all` feature enables them together.
- `src/lib.rs`: Crate root with module declarations, crate-level docs, and API conventions.
- `pygraphina/`: PyGraphina, the Python bindings crate built with maturin and published to PyPI as `pygraphina`. Contains its own `Cargo.toml`,
  `src/`, `tests/`, a `pygraphina/` type-stub package (`__init__.pyi` plus one `.pyi` per submodule, with `py.typed`), and docs.
- `benches/`: Criterion micro-benchmarks (`graph_benchmarks`, `algorithm_benchmarks`, `project_benchmarks`) that track Graphina's own performance over
  time, run by `make bench`.
- `comparisons/`: standalone comparison harnesses that measure Graphina against other libraries: `comparisons/graphina` (versus rustworkx-core) and
  `comparisons/pygraphina` (versus rustworkx and NetworkX), run by `make bench-graphina` and `make bench-pygraphina`.
- `tests/`: Workspace integration, end-to-end, regression, and property-based tests, plus `tests/testdata/` (downloaded via `make testdata`).
- `docs/`, `mkdocs.yml`: MkDocs documentation site.
- `Makefile`: GNU Make wrapper around `cargo`, maturin, and tooling commands.
- `rust-toolchain.toml`: Pinned Rust toolchain (1.85.0 as MSRV) with `rustfmt` and `clippy`.

## Architecture

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [habedi/graphina](https://github.com/habedi/graphina) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
