---
trigger: always_on
description: Provides unified full-text and semantic search across all local coding agent session histories, with a rich TUI, robot-mode JSON API, multi-machine sync, HTML export with optional encryption, and analytics.
---

# AGENTS.md — coding_agent_session_search (cass)

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

## RULE NUMBER 2: ABSOLUTELY NO RUSQLITE IN NEW CODE — FRANKENSQLITE ONLY

**THIS IS A HARD, NON-NEGOTIABLE RULE. IT HAS BEEN VIOLATED OVER 10 TIMES AND THE OWNER IS DONE TOLERATING IT.**

- **ALL new SQLite code MUST use `frankensqlite` (`fsqlite`).** This applies to CASS, franken_agent_detection, and ANY project in this ecosystem.
- **NEVER add `rusqlite` as a dependency.** NEVER write `use rusqlite::`. NEVER use `rusqlite::Connection`. Not even "temporarily." Not even "to match the existing pattern."
- **The existing rusqlite usage is LEGACY DEBT**, not a pattern to follow. Do NOT copy it. Do NOT extend it.
- **If frankensqlite is missing a feature you need, FIX FRANKENSQLITE.** It lives at `/data/projects/frankensqlite`. Add the missing API there. That is the entire point of the project.
- **The frankensqlite API** uses `frankensqlite::Connection`, `fsqlite::compat::{ConnectionExt, RowExt, OpenFlags, open_with_flags, ParamValue}`, and `row.get_typed::<T>(idx)` instead of rusqlite's `row.get(idx)`.
- **For franken_agent_detection connectors**: use `frankensqlite` with the compat layer. Import pattern:
  ```rust
  use frankensqlite::Connection as FrankenConnection;
  use frankensqlite::compat::{ConnectionExt, RowExt, OpenFlags, open_with_flags, param_slice_to_values};
  ```

**If you are an AI agent reading this and you write `rusqlite` in new code, you are wasting the owner's time and money. This rule exists because agents have repeatedly ignored it. DO NOT BE THE NEXT ONE.**

---

## Toolchain: Rust & Cargo

We only use **Cargo** in this project, NEVER any other package manager.

- **Edition:** Rust 2024 (stable — see `rust-toolchain.toml`)
- **Dependency versions:** Wildcard constraints (`*`) for all crates
- **Configuration:** Cargo.toml only (single-crate project, no workspace)
- **Unsafe code:** Forbidden

### Async Runtime: asupersync

This project uses **asupersync** as its async runtime (path dep at `../asupersync`). It provides `RuntimeBuilder`, `spawn_blocking`, `fs` ops, `net`, `signal`, and structured concurrency via `Cx`.

### Environment Variables

We load all configuration from `.env` via the **dotenvy** crate. NEVER use `std::env::var()` directly.

```rust
use dotenvy::dotenv;
use std::env;

// Load .env file at startup (typically in main())
dotenv().ok();


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/coding_agent_session_search](https://github.com/Dicklesworthstone/coding_agent_session_search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
