---
trigger: always_on
description: This file provides guidance to LLM agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to LLM agents when working with code in this repository.

## Project Overview

`suiseki` is a read-only Bun/TypeScript CLI for rendering diffs and code in the terminal.

The product starts as a unified-view `git diff` renderer and grows into a broader terminal code viewer with `view` and `tree` subcommands. It combines Pierre's diff parsing model with Shiki syntax tokenization, then emits ANSI directly so syntax foreground colors and diff background colors can coexist cleanly.

Position `suiseki` as a friendly terminal surface for Pierre's renderer-agnostic packages and Shiki's syntax/theme ecosystem: `@pierre/diffs` first, `@pierre/trees` next, and Shiki throughout. It is a homage and companion, not a fork or replacement.

Core constraints:
- Act as a Unix filter: read input, write ANSI to stdout, exit.
- Never modify user files.
- Compile to a single local `suiseki` binary with Bun.
- Stay in the TypeScript/Bun ecosystem; do not introduce Go tooling.
- Do not build interactive terminal UI behavior unless the project's direction explicitly changes.
- Keep normal `stdout` clean for rendered output. Send errors and diagnostics to `stderr`, and do not introduce service-style logging dependencies such as `pino` unless there is a concrete need.

## Essential Commands

All commands are available via `make`. Run `make` or `make help` to see the full list.

### Development
- `make run` - Run as TypeScript
- `make build` - Build `./bin/suiseki` with `bun build --compile`
- `make start` - Run `./bin/suiseki`
- `make clean` - Remove compiled `bin/` and release `dist/` artifacts

### Testing
- `make test` - Run all tests with coverage

### Code Quality
- `make check` - Run TypeScript type check and Biome linting/formatting
- `make format` - Format code with Biome
- `make check-ci` - CI version of checks (no auto-fix)

## Architecture

### Tech Stack
- **Runtime**: Bun, TypeScript
- **Binary output**: `bun build --compile`
- **Syntax highlighting**: Shiki tokenization and Shiki-compatible themes
- **Diff parsing**: `@pierre/diffs` parsing/iteration utilities
- **ANSI output**: direct ANSI emission, with `ansis` available for helpers
- **Config**: `smol-toml` for TOML config parsing
- **Validation**: Arktype (TypeScript-first runtime validation)
- **Testing**: Bun test
- **Tooling**: Biome (formatting/linting)

## Development Patterns

### Code Consistency
- **Study existing patterns first** - Review similar code to maintain consistency
- **When renaming files, use `git mv`** - Preserves git history

### Git Guidelines
- **Never use `git -C`** — always assume you're in the correct working directory
- Don't mention that code was generated or co-authored by an LLM agent
- Before committing, run `bun check` to apply formatting
- When formatting is the only change, use commit message "fmt"


##### Validator + type naming convention
Define Arktype schemas as `vFoo` and derive named TypeScript types as `Foo` directly below them:
```typescript
export const vSuisekiConfig = type({
	theme: "string",
	view: "'unified' | 'split'",
})
export type SuisekiConfig = typeof vSuisekiConfig.infer
```

##### Runtime validation boundaries
Use Arktype for data crossing runtime boundaries: config files, environment variables, CLI options, theme metadata, and parsed external JSON. Avoid `as` casts at those boundaries; parse and validate first.

##### Shared validators
When reusable validators become necessary, keep them in `src/common/validators.ts` and prefer shared schemas over duplicated inline string validators.

### Testing Guidelines

#### Test File Naming & Imports
- `*.test.ts` - Unit tests (fast, no external dependencies)
- `*.integration.test.ts` - Integration tests that exercise the compiled CLI, filesystem fixtures, or git subprocess behavior
- Both use `import { test, expect, describe } from "bun:test"`

#### What NOT to Test
- Do not snapshot entire ANSI outputs when a smaller fixture or focused assertion can cover the behavior.
- Do not test Shiki, Pierre, Biome, Bun, or Arktype internals.
- Do not add tests for planned features before implementation exists.
- Do not rely on global git/user configuration in tests.

#### What TO Test
- Diff rendering behavior, including file headers, hunk headers, gutters, signs, and ANSI reset correctness.
- Config resolution and Arktype validation.
- CLI input selection: stdin versus git subprocess arguments.
- Edge-case fixtures for binary files, renames, empty diffs, large diffs, and merge conflicts as those features land.

#### Test Structure
- **Describe block per method/function** — group related tests under `describe("methodName", () => { ... })`
- **Classes** — use the class name as the top-level describe, with nested describes per method:
  ```typescript
  describe("ConfigLoader", () => {
    describe("load", () => { ... })
    describe("resolve", () => { ... })
  })
  ```
- **Files with multiple exported functions** — use the filename as the top-level describe so you can run one describe to cover the entire file:
  ```typescript
  // ansi.test.ts
  describe("ansi.ts", () => {
    describe("emitToken", () => { ... })
    describe("emitLine", () => { ... })
    describe("emitReset", () => { ... })
  })
  ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [michaeldlfx/suiseki](https://github.com/michaeldlfx/suiseki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
