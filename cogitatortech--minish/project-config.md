---
trigger: always_on
description: This file provides guidance to coding agents collaborating on this repository.
---

# AGENTS.md

This file provides guidance to coding agents collaborating on this repository.

## Mission

Minish is a small property-based testing framework for Zig, inspired by
[QuickCheck](https://hackage.haskell.org/package/QuickCheck) and
[Hypothesis](https://hypothesis.readthedocs.io/en/latest/).
It generates random inputs, checks user-defined properties, and shrinks failing cases to a minimal reproducer.
Minish is designed to be embedded in Zig projects as a pure-Zig library.
Priorities, in order:

1. Correctness of generators, combinators, and shrinking.
2. Minimal public API for use as a library from other Zig projects.
3. Zero non-Zig dependencies and maintainable, well-tested code.
4. Cross-platform support (Linux, macOS, and Windows).

## Core Rules

- Use English for code, comments, docs, and tests.
- Prefer small, focused changes over large refactoring.
- Add comments only when they clarify non-obvious behavior.
- Do not add features, error handling, or abstractions beyond what is needed for the current task.
- Keep the project dependency-free: no external Zig packages or C libraries unless explicitly agreed.

## Writing Style

- Use Oxford commas in inline lists: "a, b, and c" not "a, b, c".
- Do not use em dashes. Restructure the sentence, or use a colon or semicolon instead.
- Avoid colorful adjectives and adverbs. Write "TCP proxy" not "lightweight TCP proxy", "scoring components" not "transparent scoring components".
- Use noun phrases for checklist items, not imperative verbs. Write "redundant index detection" not "detect redundant indexes".
- Headings in Markdown files must be in the title case: "Build from Source" not "Build from source". Minor words (a, an, the, and, but, or, for, in,
  on, at, to, by, of, is, are, was, were, be) stay lowercase unless they are the first word.

## Repository Layout

- `src/lib.zig`: Public API entry point. Re-exports `gen`, `combinators`, `check`, `Options`, `TestCase`, and `GenError`.
- `src/minish/core.zig`: Core types (`TestCase`, `GenError`) shared by generators, combinators, and the runner.
- `src/minish/gen.zig`: Built-in generators (integers, floats, booleans, characters, strings, lists, arrays, tuples, structs, optionals, hashmaps,
  UUIDs, timestamps, etc.).
- `src/minish/combinators.zig`: Generator combinators (`map`, `flatMap`, `filter`, `oneOf`, `frequency`, `dependent`, `sized`).
- `src/minish/runner.zig`: Test runner. Defines `check` and `Options` (num_runs, seed, verbose, max shrink attempts).
- `src/minish/shrink.zig`: Shrinking strategies for integers, floats, strings, lists, tuples, arrays, and optionals.
- `examples/`: Self-contained example programs (`e1_simple_example.zig` through `e8_misc_features.zig`) built as executables via `build.zig`.
- `.github/workflows/`: CI workflows (`tests.yml` for unit tests on Linux and Windows, `docs.yml` for API doc deployment).
- `build.zig` / `build.zig.zon`: Zig build configuration and package metadata.
- `Makefile`: GNU Make wrapper around `zig build` targets.
- `docs/`: Generated API docs land in `docs/api/` (produced by `make docs`).

## Architecture

### Property-Test Pipeline

A property test flows through: Generator (`gen.zig` / `combinators.zig`) -> `TestCase` (`core.zig`) ->
user property function -> Shrinker (`shrink.zig`) on failure -> reproducer report. `check` in `runner.zig` ties these together and owns the RNG seed,
run count, and verbosity.

### Generators and Combinators Split

- `src/minish/gen.zig` contains the built-in generators for primitive and collection types.
- `src/minish/combinators.zig` contains functions to compose and transform generators.
- New generators should be added to `gen.zig`. New composition primitives go in `combinators.zig`.

### Shrinking

Shrinking is the process of reducing a failing input to a smaller one that still reproduces the failure.
Each shrinkable type lives alongside its shrinker logic in `shrink.zig`.
When adding a new generator for a composite type, consider whether a shrinker is needed and add it to `shrink.zig`.

### Public API Surface

Everything re-exported from `src/lib.zig` is part of the public API.
Changes to names or signatures there are breaking.
The rest of `src/minish/` is internal and may be refactored freely as long as the public surface and its behavior are preserved.

### Dependencies

Minish has **no external Zig or C dependencies**.
The only `build.zig.zon` entries should be Minish itself.
Please do not add dependencies without prior discussion.

## Zig Conventions

- Zig version: 0.16.0 (as declared in `build.zig.zon` and the Makefile's `ZIG_LOCAL` path).
- Formatting is enforced by `zig fmt`. Run `make format` before committing.
- Naming follows Zig standard-library conventions: `camelCase` for functions (e.g. `intRange`, `flatMap`, `oneOf`), `snake_case` for local variables
  and struct fields, `PascalCase` for types and structs, and `SCREAMING_SNAKE_CASE` for top-level compile-time constants.

## Required Validation

Run the relevant targets for any change:

| Target         | Command                              | What It Runs                                                     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CogitatorTech/minish](https://github.com/CogitatorTech/minish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
