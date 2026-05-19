---
trigger: always_on
description: provides default comparators for both numeric and string key types; users can supply custom comparators via `bptree_create`.
---

# AGENTS.md

This file provides guidance to coding agents collaborating on this repository.

## Mission

Bptree is a lightweight, single-header B+ tree implementation in C.
It provides an in-memory ordered map where keys are stored in sorted order and values can be any type.
The library is designed to be embedded in other C projects by including a single header file.

Priorities, in order:

1. Correctness of tree operations (insertion, deletion, search, and range queries) and structural invariants.
2. Minimal public API surface for use as a drop-in library from other C projects.
3. Zero external dependencies and maintainable, well-tested code.
4. Memory safety: no leaks, no undefined behavior, no buffer overflows.

## Core Rules

- Use English for code, comments, docs, and tests.
- Prefer small, focused changes over large refactoring.
- Add comments only when they clarify non-obvious behavior.
- Do not add features, error handling, or abstractions beyond what is needed for the current task.
- Keep the project dependency-free: no external C libraries unless explicitly agreed. The only dependencies are the C11 standard library headers.

## Writing Style

- Use Oxford commas in inline lists: "a, b, and c" not "a, b, c".
- Do not use em dashes. Restructure the sentence, or use a colon or semicolon instead.
- Avoid colorful adjectives and adverbs. Write "B+ tree" not "lightweight B+ tree", "range query" not "efficient range query".
- Use noun phrases for checklist items, not imperative verbs. Write "node merge logic" not "merge nodes".
- Headings in Markdown files must be in the title case: "Build from Source" not "Build from source". Minor words (a, an, the, and, but, or, for, in,
  on, at, to, by, of, is, are, was, were, be) stay lowercase unless they are the first word.

## Repository Layout

- `include/bptree.h`: Single-header library. Public API declarations are at the top; the implementation is guarded by
  `#ifdef BPTREE_IMPLEMENTATION`. Internal types (`bptree`, `bptree_node`) are defined only inside that guard;
  external consumers must treat them as opaque and access tree state through the public accessor functions.
- `test/test_bptree.c`: Unit test suite using a custom test harness (`ASSERT`, `RUN_TEST` macros).
- `test/bench_bptree.c`: Performance benchmarks for insertions, searches, deletions, iteration, and range queries.
- `test/example.c`: Example usage demonstrating tree creation, CRUD operations, range queries, and custom comparators.
- `test/extras/`: Additional test files for specific scenarios (merge operations, bug reproductions, string keys).
- `.github/workflows/`: CI workflows (`tests.yml` for unit tests and coverage, `lints.yml` for static analysis, `benches.yml`
  for benchmarks).
- `Makefile`: GNU Make build automation with targets for building, testing, linting, formatting, profiling, and Zig build
  integration.
- `build.zig` / `build.zig.zon`: Zig build system configuration (requires Zig 0.16.0). Compiles the same C source files as the
  Makefile.
- `.clang-format`: Code formatting rules (Google base style, 4-space indent, 100-column limit).
- `Doxyfile`: Doxygen configuration for API documentation generation.

## Architecture

### Single-Header Design

The entire library lives in `include/bptree.h`. Users include the header normally for declarations, and define
`BPTREE_IMPLEMENTATION` in exactly one translation unit to pull in the implementation. This keeps integration simple: copy one
file, add two lines of code.

The public section (before the `#ifdef BPTREE_IMPLEMENTATION` guard) includes only `<stdbool.h>` and `<stdint.h>`.
Implementation-only headers (`assert.h`, `stdalign.h`, `stdarg.h`, `stdio.h`, `stdlib.h`, `string.h`, `time.h`) are included
inside the guard to avoid polluting the consumer's namespace. `bptree_node` is an opaque type in the public section (forward
declaration only); its full definition is inside the implementation guard.

### Key and Value Generics

Key and value types are configured at compile time via preprocessor macros (`BPTREE_NUMERIC_TYPE`,
`BPTREE_KEY_TYPE_STRING`, `BPTREE_KEY_SIZE`, `BPTREE_VALUE_TYPE`). Defaults are `int64_t` keys and `void *` values. The library
provides default comparators for both numeric and string key types; users can supply custom comparators via `bptree_create`.

### Tree Operations

- **Insertion** (`bptree_put`): inserts a key-value pair, splitting nodes as needed. Duplicate keys are rejected.
- **Search** (`bptree_get`, `bptree_contains`): traverses internal nodes to find the correct leaf.
- **Deletion** (`bptree_remove`): removes a key-value pair and rebalances the tree via key borrowing or node merging.
- **Range query** (`bptree_get_range`): returns all values in a `[start, end]` inclusive range by scanning leaf nodes.
- **Iteration** (`bptree_iter_*`): forward iterator over the leaf-level linked list. Supports `begin`, `next`, `find`,
  `lower_bound`, and `upper_bound`.
- **Accessors** (`bptree_count`, `bptree_height`): O(1) reads. `bptree_clear` resets the tree without freeing it.

### Node Layout

Nodes use flexible array members (FAM) for memory-efficient storage with proper alignment. Internal nodes store keys and child

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [habedi/bptree](https://github.com/habedi/bptree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
