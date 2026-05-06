---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust + Bash codebase.
---

# AGENTS.md — process_triage

> Guidelines for AI coding agents working in this Rust + Bash codebase.

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

## Toolchain: Rust & Cargo + Bash

This project uses **Cargo** for the Rust workspace and **Bash** for the wrapper script and installer.

- **Edition:** Rust 2021 (nightly required — see `rust-toolchain.toml`)
- **MSRV:** 1.88 (raised for `toon` edition 2024 + vergen-gix dependency)
- **Dependency versions:** Explicit versions for stability
- **Configuration:** Cargo.toml workspace with `workspace = true` pattern
- **Bash style:** `shellcheck`-clean, prefer `[[` over `[`, quote variables, use `local`, prefer `printf` over `echo`

### Key Dependencies

| Crate | Purpose |
|-------|---------|
| `clap` | CLI argument parsing with derive macros |
| `serde` + `serde_json` | Serialization |
| `chrono` | Timestamps with serde support |
| `uuid` | v4 UUIDs with serde support |
| `thiserror` | Ergonomic error type derivation |
| `tracing` + `tracing-subscriber` | Structured logging and diagnostics |
| `toon` | TOON structured output format |
| `schemars` | JSON Schema generation |
| `regex` | Pattern matching for process heuristics |
| `sha2` + `hex` | Cryptographic hashing |
| `p256` + `base64` | ECDSA signature verification for release binaries |
| `ftui` | Premium TUI experience (Elm-style, behind `ui` feature) |
| `ftui-harness` | Snapshot test harness for TUI golden tests |
| `arrow` + `parquet` | Telemetry storage (Apache Arrow schemas, Parquet writer) |
| `chacha20poly1305` + `pbkdf2` | Bundle encryption |
| `zip` | Session bundle format (ZIP with deflate) |
| `prometheus` + `tiny_http` | Metrics endpoint (behind `metrics` feature) |
| `askama` + `minify-html` | HTML report templating and minification |
| `criterion` | Benchmarking |
| `proptest` | Property-based testing |

### Release Profile

The release build optimizes for size (used for musl distribution builds):

```toml
[profile.release]
lto = "fat"             # Full link-time optimization
codegen-units = 1       # Single codegen unit for better optimization
strip = true            # Remove debug symbols
panic = "abort"         # Smaller binary, no unwinding

[profile.release-small]
inherits = "release"
opt-level = "s"         # Size-optimized for distribution
```

---

## Code Editing Discipline

### No Script-Based Changes

**NEVER** run a script that processes/changes code files in this repo. Brittle regex-based transformations create far more problems than they solve.

- **Always make code changes manually**, even when there are many instances

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/process_triage](https://github.com/Dicklesworthstone/process_triage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
