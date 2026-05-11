---
trigger: always_on
description: <!-- agent-pmo:3140e31 -->
---

<!-- agent-pmo:3140e31 -->
# dart_mutant — Agent Instructions

⚠️ CRITICAL: **Reduce token usage.** Check file size before loading. Write less. Delete fluff and dead/duplicate code/docs. ⚠️

> Read this entire file before writing any code.
> These rules are NON-NEGOTIABLE. Violations will be rejected in review.

## Project Overview

`dart_mutant` is a Rust-based mutation testing tool for Dart. It uses tree-sitter
for AST-based mutations, generates syntactically valid mutants, runs `dart test`
against each mutant in parallel, and produces HTML (dark theme) and JSON
(Stryker-compatible) reports so you can see which mutations escaped your test suite.

**Primary language:** Rust (edition 2021)
**Repo type:** CLI tool
**Build command:** `make ci`
**Test command:** `make test`
**Lint command:** `make lint`

## Too Many Cooks (Multi-Agent Coordination)

If the TMC server is available:
1. Register immediately: descriptive name, intent, files you will touch
2. Before editing any file: lock it via TMC
3. Broadcast your plan before starting work
4. Check messages every few minutes
5. Release locks immediately when done
6. Never edit a locked file — wait or find another approach

## Hard Rules — Universal (no exceptions)

- **DO NOT use git commands.** No `git add`, `git commit`, `git push`, `git checkout`, `git merge`, `git rebase`, or any other git command. CI and GitHub Actions handle git.
- **ZERO DUPLICATION.** Before writing any code, search the codebase for existing implementations. Move code, don't copy it.
- **NO THROWING / PANICKING.** Return `Result<T, E>` or `Option<T>`. Panics are only for unrecoverable bugs — and they must never reach production code paths.
- **NO REGEX on structured data.** Never parse JSON, YAML, TOML, code, or any structured format with regex. Use `serde_json`, `toml`, or tree-sitter.
- **NO PLACEHOLDERS.** If something isn't implemented, leave a loud compilation error (`todo!()`). Never write code that silently does nothing.
- **Functions < 20 lines.** Refactor aggressively. If a function exceeds 20 lines, split it.
- **Files < 500 lines.** If a file exceeds 500 lines, extract modules.
- **100% test coverage is the goal.** Never delete or skip tests. Never remove assertions.
- **`make test` is FAIL-FAST.** Stops at first failing test. Never `--no-fail-fast`. Saves CI minutes.
- **`make test` ALWAYS computes coverage AND enforces it.** Threshold lives in `coverage-thresholds.json` — NOT env vars, NOT GitHub repo variables. Below threshold = pipeline fails. Ratchet only.
- **Prefer E2E/integration tests.** Unit tests are acceptable only for isolating problems.
- **Heavy logging everywhere.** Use `tracing` (already in `Cargo.toml`). See Logging Standards.
- **No suppressing linter warnings.** Fix the code, not the linter.
- **Pure functions** over statements.
- **Every spec section MUST have a unique, hierarchical, non-numeric ID.** Format: `[GROUP-TOPIC]` or `[GROUP-TOPIC-DETAIL]` (e.g., `[PARSER-DART-AST]`, `[RUNNER-TIMEOUT]`). The first word is the **group** — all sections in the same group MUST be adjacent in the spec's TOC. NEVER use sequential numbers like `[SPEC-001]`. All code, tests, and design docs that implement or relate to a spec section MUST reference its ID in a comment (e.g., `// Implements [RUNNER-TIMEOUT]`).

## Logging Standards

- **Use `tracing`.** Never use `println!`, `eprintln!`, `dbg!`, or `print!` for diagnostics. The `tracing` crate is already a dependency; pair it with `tracing-subscriber` for output.
- **Log at entry/exit of all significant operations.** Use levels: `error`, `warn`, `info`, `debug`, `trace`.
- **Logging must be throughout the app.** Every non-trivial operation (parsing, mutation application, test run, report generation) should log. Silent failures are forbidden.
- **NEVER log personal data.** No names, emails, addresses, phone numbers, IPs, or any PII.
- **NEVER log secrets.** No API keys, tokens, passwords, connection strings. If you need to confirm an AI-provider key is loaded, log `"API key: present"`.
- **Structured fields over string interpolation.** Use `tracing::info!(file = %path, mutations = count, "discovered mutations")`, not `info!("Found {} mutations in {}", count, path)`.

| Language | Library | Notes |
|----------|---------|-------|
| Rust | `tracing` | With `tracing-subscriber` for output |

## Hard Rules — Rust

- No `unwrap()` — use `?` or explicit `match`
- No `expect()` in production code (tests may use it)
- No `panic!()`, `todo!()`, `unimplemented!()`, `unreachable!()` in production code
- No `unsafe {}` blocks (`unsafe_code = "deny"` in `Cargo.toml`)
- No `#[allow(clippy::...)]` attributes without a documented justification
- All public items must have doc comments (`///`) — `missing_docs = "deny"`
- Use `thiserror` for error types; `anyhow::Result` is acceptable in `main.rs`/CLI code
- All Clippy lints run at `deny` level: `all`, `pedantic`, `nursery`, `cargo`
- Prefer iterator chains (`map`, `filter`, `and_then`, `collect`) over imperative loops
- Use `?` for error propagation — no manual `match err { ... return Err(...) }` ladders

## Testing Rules

- **Never delete a failing test.** Fix the code or fix the test expectation — never delete.
- **Never skip a test** (`#[ignore]`) without a ticket number and expiry date in the skip reason.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nimblesite/dart_mutant](https://github.com/Nimblesite/dart_mutant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
