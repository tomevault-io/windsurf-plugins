---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust codebase.
---

# AGENTS.md — vibe_cockpit

> Guidelines for AI coding agents working in this Rust codebase.

---

## RULE 0 - THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

---

## Irreversible Git & Filesystem Actions — DO NOT EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. "I think it's safe" is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it—if anything remains ambiguous, refuse and escalate.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time. If that record is absent, the operation did not happen.

---

## Git Branch: ONLY Use `main`, NEVER `master`

**The default branch is `main`. The `master` branch exists only for legacy URL compatibility.**

- **All work happens on `main`** — commits, PRs, feature branches all merge to `main`
- **Never reference `master` in code or docs** — if you see `master` anywhere, it's a bug that needs fixing
- **The `master` branch must stay synchronized with `main`** — after pushing to `main`, also push to `master`:
  ```bash
  git push origin main:master
  ```

**If you see `master` referenced anywhere:**
1. Update it to `main`
2. Ensure `master` is synchronized: `git push origin main:master`

---

## Toolchain: Rust & Cargo

We only use **Cargo** in this project, NEVER any other package manager.

- **Edition:** Rust 2024 (nightly required)
- **Dependency versions:** Explicit versions for stability
- **Configuration:** Cargo.toml workspace with `workspace = true` pattern
- **Unsafe code:** Forbidden (`#![forbid(unsafe_code)]`)

### Async Runtime: Asupersync (primary) + Tokio compat bridge

This project uses **Asupersync** (`/dp/asupersync`) as the primary async runtime with explicit `Cx` capability tokens for structured concurrency. A **Tokio compat bridge** (`asupersync-tokio-compat`) keeps tokio-locked crates (axum, russh, reqwest) working until native alternatives are ready. The binary entry point (`src/main.rs`) builds an Asupersync runtime, establishes a root `Cx`, and runs the CLI inside `with_tokio_context()`.

### Key Dependencies

| Crate | Purpose |
|-------|---------|
| `asupersync` | Primary async runtime (structured concurrency, Cx tokens) |
| `asupersync-tokio-compat` | Tokio compatibility bridge for downstream crates |
| `tokio` | Compat bridge runtime (axum, russh, reqwest still use tokio internally) |
| `duckdb` | Embedded analytical database (bundled build) |
| `serde` + `serde_json` + `toml` | Serialization (JSON, TOML config) |
| `clap` | CLI argument parsing (derive mode) |
| `ftui` | Terminal UI rendering (FrankenTUI Elm-architecture framework) |
| `axum` + `tower` + `tower-http` | Web server, middleware, CORS, static files, tracing |
| `chrono` | Date/time handling with serde support |
| `thiserror` + `anyhow` | Error type derivation and ad-hoc errors |
| `tracing` + `tracing-subscriber` | Structured logging with env-filter and JSON output |
| `reqwest` | HTTP client for webhook delivery and external APIs |
| `russh` + `russh-keys` | SSH client for remote machine data collection |
| `uuid` | Unique IDs (v4, serde-compatible) |
| `regex` | Pattern matching in alert conditions |
| `async-trait` | Async trait support for collector and alert channel interfaces |
| `dashmap` | Concurrent hash maps for alert cooldown tracking |
| `futures` | Async stream and utility combinators |
| `rand` | Random number generation for oracle predictions |
| `proptest` | Property-based testing (dev) |
| `mockall` | Trait mocking for unit tests (dev) |
| `vergen-gix` | Build metadata embedding (build.rs) |

### Release Profile

The release build optimizes for performance:

```toml

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/vibe_cockpit](https://github.com/Dicklesworthstone/vibe_cockpit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
