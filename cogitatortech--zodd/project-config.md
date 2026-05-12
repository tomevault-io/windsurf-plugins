---
trigger: always_on
description: This file provides guidance to coding agents collaborating on this repository.
---

# AGENTS.md

This file provides guidance to coding agents collaborating on this repository.

## Mission

Zodd is a small, embeddable [Datalog](https://en.wikipedia.org/wiki/Datalog) engine written in pure Zig.
It evaluates recursive rules over sets of tuples using semi-naive iteration, merge joins, and indexed extension primitives.
Zodd is designed to be embedded in Zig projects as a library.
Priorities, in order:

1. Correctness of relations, variables, joins, extensions, and fixed-point iteration.
2. Minimal public API for use as a library from other Zig projects.
3. Small dependency footprint and maintainable, well-tested code.
4. Cross-platform support (Linux, macOS, and Windows).

## Core Rules

- Use English for code, comments, docs, and tests.
- Prefer small, focused changes over large refactoring.
- Add comments only when they clarify non-obvious behavior.
- Do not add features, error handling, or abstractions beyond what is needed for the current task.
- Keep the dependency set small: do not add new Zig packages or C libraries without prior discussion.

## Writing Style

- Use Oxford commas in inline lists: "a, b, and c" not "a, b, c".
- Do not use em dashes. Restructure the sentence, or use a colon or semicolon instead.
- Avoid colorful adjectives and adverbs. Write "Datalog engine" not "blazing-fast Datalog engine", "merge join" not "efficient merge join".
- Use noun phrases for checklist items, not imperative verbs. Write "redundant index detection" not "detect redundant indexes".
- Headings in Markdown files must be in the title case: "Build from Source" not "Build from source". Minor words (a, an, the, and, but, or, for, in,
  on, at, to, by, of, is, are, was, were, be) stay lowercase unless they are the first word.

## Repository Layout

- `src/lib.zig`: Public API entry point. Re-exports `Relation`, `Variable`, `Iteration`, `ExecutionContext`, join helpers, and extend primitives.
- `src/zodd/relation.zig`: Immutable `Relation` type (sorted, deduplicated tuples).
- `src/zodd/variable.zig`: Mutable `Variable` type for fixed-point iteration, plus the `gallop` search helper.
- `src/zodd/iteration.zig`: `Iteration` driver for semi-naive evaluation.
- `src/zodd/join.zig`: Merge-join algorithms (`joinHelper`, `joinInto`, `joinAnti`).
- `src/zodd/extend.zig`: Leaper-based extension primitives (`ExtendWith`, `FilterAnti`, `ExtendAnti`, `extendInto`).
- `src/zodd/index.zig`: Indexes for keyed lookups.
- `src/zodd/aggregate.zig`: Group-by and aggregation operations.
- `tests/`: Non-unit tests (`integration_tests.zig`, `regression_tests.zig`, `property_tests.zig`, `incremental_tests.zig`).
- `examples/`: Self-contained example programs (`e1_network_reachability.zig` through `e6_dependency_resolution.zig`) built as executables via
  `build.zig`.
- `.github/workflows/`: CI workflows (`tests.yml` for unit and integration tests, `docs.yml` for API doc deployment).
- `build.zig` / `build.zig.zon`: Zig build configuration and package metadata.
- `Makefile`: GNU Make wrapper around `zig build` targets.
- `docs/`: Generated API docs land in `docs/api/` (produced by `make docs`).

## Architecture

### Evaluation Pipeline

A Datalog program flows through: base data is loaded into a `Relation` (`relation.zig`). Derived predicates use a `Variable` (`variable.zig`) driven
by an `Iteration` (`iteration.zig`) loop that calls `changed()` until a fixed point. Each iteration extends tuples via `join` (`join.zig`) or `extend`
(`extend.zig`), optionally using indexes (`index.zig`) or aggregates (`aggregate.zig`). Every primitive takes a `std.mem.Allocator` directly; there is
no wrapper context type.

### Relations and Variables Split

- `relation.zig` is the immutable, sorted, deduplicated tuple container used for base facts and finalized results.
- `variable.zig` is the mutable counterpart used inside fixed-point loops; it tracks stable, recent, and to-add tuple sets for semi-naive evaluation.
- New join shapes go in `join.zig`. New leaper-style extensions go in `extend.zig`.

### Indexing and Aggregation

`index.zig` provides keyed lookups used by the extend primitives. `aggregate.zig` provides group-by reductions.
When adding a new join or extension shape, consider whether it needs an index variant and add it alongside the existing ones.

### Public API Surface

Everything re-exported from `src/lib.zig` is part of the public API.
Changes to names or signatures there are breaking.
The rest of `src/zodd/` is internal and may be refactored freely as long as the public surface and its behavior are preserved.

### Dependencies

Zodd depends on two sibling Zig packages declared in `build.zig.zon`:

- `ordered`: sorted container primitives, linked into the `zodd` module for all builds.
- `minish`: property-testing framework, used only by `tests/property_tests.zig` and lazy-loaded in `build.zig`.

Please do not add further dependencies without prior discussion.

## Zig Conventions

- Zig version: 0.16.0 (as declared in `build.zig.zon` and the Makefile's `ZIG_LOCAL` path). CI pins the version declared in `build.zig.zon`.
- Formatting is enforced by `zig fmt`. Run `make format` before committing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CogitatorTech/zodd](https://github.com/CogitatorTech/zodd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
