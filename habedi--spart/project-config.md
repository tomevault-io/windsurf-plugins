---
trigger: always_on
description: This file provides guidance to coding agents collaborating on this repository.
---

# AGENTS.md

This file provides guidance to coding agents collaborating on this repository.

## Mission

Spart is a collection of space partitioning tree data structures for Rust, with Python bindings.
Priorities, in order:

1. Correct query behavior: a query must return every point it covers, not merely correct ones, and every tree must hold its structural invariants
   after each operation.
2. Clear boundaries between the shared geometric primitives, the individual tree implementations, and the Python bindings.
3. Reproducible, benchmark-backed performance; no premature optimization before correctness is covered.
4. Idiomatic Rust: ownership, zero-cost abstractions, and `unsafe` only where necessary and documented.

## Core Rules

- Use English for code, comments, docs, and tests.
- Prefer small, focused changes over broad rewrites.
- Keep the module boundaries: `geometry` owns the primitives and the traits over them, each tree module owns one data structure, `rtree_common` owns
  the logic shared by the two R-tree variants, and `pyspart` wraps the public API. Do not import across those boundaries in the wrong direction; see
  Dependency Boundaries.
- Keep all mutable state inside the tree values themselves; do not introduce module-level `static mut`, `lazy_static`, or `OnceLock` globals for
  runtime state. The crate touches no process-global state at all, and installs no tracing subscriber: choosing one is the application's decision.
- `#![forbid(unsafe_code)]` is set in `src/lib.rs`. Nothing here needs `unsafe`; if you reach for it, the design is wrong.
- A tree operation that cannot store a point must report it. Never drop a point silently; return `false`, return a count, or return an `Err`.
- Define a concept once. Nearest-neighbor accumulation lives in `knn::KnnHeap`, minimum distance in `HasMinDistance`, and the indexable-object
  contract in `BoundedObject`. If you find yourself writing a second copy for another dimension or another tree variant, reach for a generic or the
  `impl_rtree_spatial_index!` macro instead.
- Do not reach for `unwrap` or `expect` in library code. `make lint` denies both. Prefer an early return, a defensive fallback that keeps data
  reachable, or a documented `unreachable!` guarded by a check on the line above.
- Respect the MSRV in `rust-toolchain.toml` and `Cargo.toml` (1.85.0). Do not use standard-library APIs stabilized later; `geometry::span` manipulates
  bits by hand rather than calling `f64::next_up` for exactly this reason.
- Add comments only when they clarify a non-obvious tree invariant, a floating-point subtlety, or why a simpler formulation is wrong.
- Maintain the permissive license boundary of the crate (MIT or Apache-2.0). Do not add dependencies or statically link libraries with copyleft, weak
  copyleft, or source-available licenses (such as GPL, MPL, or SSPL).
- Format with `rustfmt` (`make format`) and lint with Clippy (`make lint`) before declaring a change done.

Quick examples:

- Good: add a `Quadtree::range_search_bbox` method next to the existing search methods, with a unit test and a brute-force comparison in
  `tests/test_completeness.rs`.
- Good: extend `geometry::BoundingVolume` with a default method and override it for `Rectangle` and `Cube`.
- Bad: duplicate the minimum-distance computation in a tree module instead of calling `HasMinDistance`.
- Bad: change a tree so a query returns a subset of the covered points, even if every returned point is correct.
- Bad: add a cargo dependency that pulls in a copyleft or source-available library.

## Writing Style

- Use Oxford commas in inline lists: "a, b, and c" not "a, b, c".
- Do not use em dashes. Restructure the sentence, or use a colon or semicolon instead.
- Avoid colorful adjectives and adverbs. Write "range query" not "blazing range query".
- Prefer noun phrases for checklist items over imperative verbs. Write "brute-force comparison" not "compare against brute force".
- Headings in Markdown files must be in title case: "Build from Source" not "Build from source". Minor words (a, an, the, and, but, or, for, in, on,
  at, to, by, of) stay lowercase unless they are the first word.
- Do not bold the lead-in of a list item. Write "Uniform depth: ..." not "**Uniform depth**: ...".
- Use sentence case for the lead-in of a list item. Write "Node splitting: ..." not "Node Splitting: ...". Proper nouns keep their capitals.
- Capitalize only the first part of a hyphenated compound: "Nearest-neighbor Search" in a heading, "Breadth-first" at the start of a sentence, and
  "breadth-first search" elsewhere. Never write "Breadth-First".
- Start each sentence with a capital letter, capitalize proper nouns (Rust, Python, Clippy, Criterion, PyO3), and leave common nouns lowercase in the
  middle of a sentence.
- Write correct and complete sentences.
- Avoid made-up words, abbreviations, and colons in the middle of sentences.
- Use participial phrases scarcely.

## Repository Layout

Do not invent modules that do not yet exist, but do place new modules according to this map.

- `src/lib.rs`: module declarations only. `errors`, `geometry`, `index`, `kdtree`, `octree`, `quadtree`, `rstar_tree`, and `rtree` are public;
  `knn` and `rtree_common` are private.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [habedi/spart](https://github.com/habedi/spart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
