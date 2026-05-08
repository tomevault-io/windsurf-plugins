---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust codebase.
---

# AGENTS.md — storage_ballast_helper (`sbh`)

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
  git push origin main main:master
  ```

**Why this matters:** Install URLs and update checks may reference `master`. If `master` falls behind `main`, users get stale code.

**If you see `master` referenced anywhere:**
1. Update it to `main`
2. Ensure `master` is synchronized: `git push origin main main:master`

---

## Multi-Agent Environment

Multiple agents work on this codebase simultaneously. This is normal and expected.

**How to handle other agents' changes:**
- You will regularly see modified files you didn't touch in `git status`. This is expected.
- **NEVER** stash, revert, overwrite, or otherwise disturb other agents' changes.
- Treat those changes identically to changes you yourself made. Commit them alongside your own work.
- If another agent's code causes compilation errors, fix them as part of your normal workflow.
- Never stop to ask about "unexpected changes" — they are always from concurrent agents.

---

## Toolchain: Rust & Cargo

We only use **Cargo** in this project, NEVER any other package manager.

- **Edition:** Rust 2024 (see `Cargo.toml`)
- **Toolchain:** Stable (see `rust-toolchain.toml`)
- **Dependency versions:** Explicit versions for stability
- **Configuration:** Cargo.toml only
- **Unsafe code:** Forbidden (`#![forbid(unsafe_code)]` in both `lib.rs` and `main.rs`)

### Key Dependencies

| Crate | Purpose |
|-------|---------|
| `clap` + `clap_complete` | CLI parsing with derive macros and shell completion generation |
| `serde` + `serde_json` + `toml` | Serialization for config (TOML), structured output (JSON) |
| `rusqlite` (bundled) | SQLite for activity logging and stats queries |
| `chrono` | Timestamps for events and log entries |
| `colored` | Terminal colors with TTY detection |
| `thiserror` | Structured error types with stable error codes |
| `parking_lot` + `crossbeam-channel` | Concurrency primitives (no tokio) |
| `memchr` | Fast byte scanning for pattern matching |
| `regex` | Artifact classification patterns |
| `sha2` | SHA-256 checksums for supply-chain verification |
| `signal-hook` | Signal handling for graceful daemon shutdown |
| `rand` | Ballast file generation and randomized test fixtures |
| `nix` + `libc` | Unix-specific filesystem and signal operations |
| `tempfile` | Test fixtures (dev-dependency) |
| `proptest` | Property-based testing (dev-dependency) |

### Release Profile

```toml
[profile.release]
opt-level = "z"     # Optimize for size (lean binary for distribution)
lto = true          # Link-time optimization
codegen-units = 1   # Single codegen unit for better optimization
panic = "abort"     # Smaller binary, no unwinding overhead

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/storage_ballast_helper](https://github.com/Dicklesworthstone/storage_ballast_helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
