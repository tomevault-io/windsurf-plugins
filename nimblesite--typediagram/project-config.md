---
trigger: always_on
description: Read this file in full. Rules below are NON-NEGOTIABLE — violations are rejected in review.
---

# typeDiagram — Agent Instructions

Read this file in full. Rules below are NON-NEGOTIABLE — violations are rejected in review.

<!-- agent-pmo:372ce7f -->

⚠️ **TOKEN ECONOMICS DISCIPLINE.** Check file size first. `Grep` over `Read`. Use `offset`/`limit`.
Smallest diff that solves the problem. Delete dead code, unused imports, stale comments.
Call out irrelevant context before proceeding. Bloat degrades reasoning. ⚠️
⚠️ ACT AUTONOMOUSLY. DON'T ASK THE USER QUESTIONS. USE YOUR JUDGEMENT. ⚠️
⚠️ NEVER KILL ANY VSCODE PROCESS ⚠️

## Project Overview

typeDiagram is a small DSL for diagramming algebraic data types (records + tagged unions). Language-neutral, no methods. Includes a parser, model, layout engine, SVG renderer, and markdown support. Ships as an npm library, CLI tool, VS Code extension, and web playground.

A **Rust workspace** under `crates/` (root `Cargo.toml`) is being introduced alongside the TypeScript monorepo. It is a multi-language repo: TypeScript config and Rust config are chained orthogonally, never merged.

## Testing Rules

### Bulk of tests

- complex diagram text- > programming language type text
- complex programming language type text example -> diagram
- complex diagram text -> SVG
- Loads of user interactions and assertions in each test
- Don't split tests for assertions or user interactions. Merge them.
- Avoid fine grained unit tests

- **Never delete or skip tests. Never remove assertions.** Fix the code or the expectation. 100% coverage is the goal.
- **Never skip a test** without a ticket number AND expiry date in the skip reason.
- **Specific assertions only.** `assert.ok(true)` is illegal.
- **No try/catch in tests that swallows exceptions and asserts success.**
- **Deterministic.** No `sleep()`, no timing dependencies, no random state.
- **E2E tests: black-box only** — public APIs, UI, or CLI. Never reach into internals.
- **VS Code extension E2E:** interact only via `vscode.commands.executeCommand`.

## Hard Rules — Universal (no exceptions)

- **Classes are illegal.** Convert classes to Haskell style type classes with typedef.
- **NO git commands.** No `add`, `commit`, `push`, `checkout`, `merge`, `rebase`, etc. CI handles git.
  If git work is ever explicitly delegated to you: NEVER push to `main` directly (always PR → CI green → merge), NEVER list yourself as a commit co-author, work on exactly ONE branch at a time (reuse the open feature branch; if several exist, merge them into one FIRST), and NEVER run `git worktree`.
- **Auto-memory is OFF** (`"autoMemoryEnabled": false` in `.claude/settings.json`). Persistent rules go through a reviewed PR to this file — never auto-captured memory.
- **ZERO DUPLICATION.** Search before writing. Move code, don't copy it. PRIORITIZE THIS OVER ALL ELSE!!
- **No throwing** — return `Result<T,E>` (framework Result type). Wrap potential failures in try/catch; return `Result<T,E>`.
- **NO PLACEHOLDERS.** If you HAVE TO leave a section blank, fail LOUDLY by throwing an exception.
- **Functions < 20 lines. Files < 500 lines.** Refactor when over. Aggressively break up violations.
- **`make test` is FAIL-FAST and enforces coverage** from `coverage-thresholds.json`. Never `--no-fail-fast`. See REPO-STANDARDS-SPEC [TEST-RULES]. Coverage only monotonically increases -1%.
- **Prefer E2E Whole-App Widget/Integration.** Unit tests only for isolating bugs. Tests must double as integration/widget tests.
- **Heavy structured logging.** Use `pino` for TypeScript — never raw `console.log`.
- **No linter suppressions.** Fix the code.
- **Pure functions over statements.**
- **Don't use if statements** — Use pattern matching or ternaries instead.
- **Spec IDs hierarchical, non-numeric: `[GROUP-TOPIC]` / `[GROUP-TOPIC-DETAIL]`** (e.g. `[AUDIO-QUEUE-PLAY]`). NO sequential numbers. Code/tests MUST reference the ID so `grep [AUDIO-` finds spec->code->tests.
- **Strict TypeScript mode enabled (`strict: true`, `noImplicitAny: true`).**
- **Never explicitly type function return values when the type is inferred** (`explicit-function-return-type` = ILLEGAL).
- **Routinely format with prettier.**
- **NO REGEX on structured data.** Use real parsers for JSON/YAML/TOML/code.
- **`make test` ALWAYS computes coverage AND enforces it.** Threshold lives in `coverage-thresholds.json` at the repo root — NOT env vars, NOT gh repo variables, NOT CI YAML. Below threshold = pipeline fails. Ratchet only. See [COVERAGE-THRESHOLDS-JSON].

## Hard Rules — TypeScript

- No `any` (use `unknown` and narrow). No `!` non-null assertion. No `// @ts-ignore`/`@ts-nocheck`.
- No implicit `any` — annotate every parameter and return type.
- No `as Type` casts without a comment explaining safety.
- `tsconfig.json` MUST have `"strict": true`.
- No throwing — return `Result<T,E>` (library or discriminated union).

## Hard Rules — Rust

Rust lives in the `crates/` workspace. Lints: `[workspace.lints]` in the root `Cargo.toml` (REPO-STANDARDS-SPEC [LINT-RUST]); formatting: `rustfmt.toml` ([LINT-RUST-FMT]).

- **Every crate inherits the workspace lints** — add `[lints]` / `workspace = true` to its `Cargo.toml`.
- **All lints ON and at `deny`.** `unsafe_code = "deny"`; clippy `all` + `pedantic` denied.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nimblesite/typeDiagram](https://github.com/Nimblesite/typeDiagram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
