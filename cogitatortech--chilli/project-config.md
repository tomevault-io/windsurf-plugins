---
trigger: always_on
description: This file provides guidance to coding agents collaborating on this repository.
---

# AGENTS.md

This file provides guidance to coding agents collaborating on this repository.

## Mission

Chilli is a command-line interface (CLI) microframework for Zig.
It turns a declarative description of commands, flags, and positional arguments into a parser, help generator, and dispatcher, with zero external
dependencies.
Priorities, in order:

1. Correctness of argument parsing, flag resolution, and help output.
2. Minimal public API for defining and running command trees from other Zig projects.
3. Zero non-Zig dependencies, maintainable, and well-tested code.
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

- `src/lib.zig`: Public API entry point. Re-exports `Command`, `CommandOptions`, `Flag`, `FlagType`, `FlagValue`, `PositionalArg`, `CommandContext`,
  `styles`, and `Error`.
- `src/chilli/command.zig`: The `Command` struct (command tree, init/deinit, `run`, subcommand and flag registration).
- `src/chilli/types.zig`: Core types (`CommandOptions`, `Flag`, `FlagType`, `FlagValue`, `PositionalArg`) and the `parseValue` helper.
- `src/chilli/parser.zig`: Argument-string parser (`ArgIterator`, `ParsedFlag`, long/short/grouped flag handling, positional handling).
- `src/chilli/context.zig`: The `CommandContext` passed to each command's `exec` function for typed flag and argument access.
- `src/chilli/errors.zig`: Error types produced by parsing and type coercion.
- `src/chilli/utils.zig`: Shared helpers (`styles` for ANSI colors, `parseBool`, and other small utilities).
- `examples/`: Self-contained example programs (`e1_simple_cli.zig` through `e8_flags_and_args.zig`) built as executables via `build.zig`.
- `.github/workflows/`: CI workflows (`tests.yml` for unit tests on Linux and Windows, `docs.yml` for API doc deployment).
- `build.zig` / `build.zig.zon`: Zig build configuration and package metadata.
- `Makefile`: GNU Make wrapper around `zig build` targets.
- `docs/`: Generated API docs land in `docs/api/` (produced by `make docs`).

## Architecture

### Command Tree

A Chilli application is a tree of `Command` nodes. Each node owns its `flags`, `positional_args`, optional `exec`
function, and a list of subcommands. `Command.init` allocates a node; `Command.deinit` recursively frees the subtree, so
downstream users call `deinit` only on the root.

### Parsing Pipeline

Arguments flow through: `ArgIterator` over `[][]const u8` (`parser.zig`) -> per-node flag and positional resolution
(`parser.zig` + `command.zig`) -> `CommandContext` population (`context.zig`) -> `exec` dispatch on the resolved leaf
command.

### Flag and Positional Types

`FlagType` (`types.zig`) enumerates the supported value kinds (`Bool`, `Int`, `Float`, `String`). `FlagValue` is the matching-tagged union.
`types.parseValue` is the single conversion point from raw strings into a `FlagValue`; every new type or coercion belongs here.

### Help and Version Output

Help and version text are generated automatically from the command tree at runtime by `command.zig`, using the metadata in `CommandOptions` (name,
description, version, sections) and the registered flags and positional args. Grouping into
named sections is supported; custom help formatting beyond that should be added sparingly.

### Public API Surface

Everything re-exported from `src/lib.zig` is part of the public API. Changes to names or signatures there are breaking.
The rest of `src/chilli/` is internal and may be refactored freely as long as the public surface and its behavior are
preserved.

### Dependencies

Chilli has **no external Zig or C dependencies**.
The only `build.zig.zon` entries should be Chilli itself.
Please do not add dependencies without prior discussion.

## Zig Conventions

- Zig version: 0.16.0 (as declared in `build.zig.zon` and the Makefile's `ZIG_LOCAL` path).
- Formatting is enforced by `zig fmt`. Run `make format` before committing.
- Naming follows Zig standard-library conventions: `camelCase` for functions (e.g. `addFlag`, `getFlag`, `parseBool`), `snake_case` for local
  variables and struct fields, `PascalCase` for types and structs, and `SCREAMING_SNAKE_CASE` for top-level compile-time constants.

## Required Validation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CogitatorTech/chilli](https://github.com/CogitatorTech/chilli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
