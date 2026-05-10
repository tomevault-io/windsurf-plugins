---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust codebase.
---

# AGENTS.md — franken_node

> Guidelines for AI coding agents working in this Rust codebase.

---

## RULE 0 - THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

---

## Repository Reality Check (Authoritative)

This repository is **franken_node** (trust-native runtime platform), not the standalone `dcg` codebase.

- **Workspace layout:** Root `Cargo.toml` is a workspace manifest.
- **Primary crate path:** `crates/franken-node/`
- **Main Rust sources:** `crates/franken-node/src/`
- **Primary binary entrypoint:** `crates/franken-node/src/main.rs`
- **Primary library exports:** `crates/franken-node/src/lib.rs`

If any later section mentions legacy `dcg` paths like `src/main.rs` at repo root, treat this section and the on-disk tree as authoritative for implementation work in this repository.

---

## Irreversible Git & Filesystem Actions — DO NOT EVER BREAK GLASS

> **Note:** This project exists specifically to block these dangerous commands for AI agents. Practice what we preach.

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

**Why this matters:** Historical installer/docs links and raw GitHub URLs can still point at `master`. If `master` falls behind `main`, users get stale `franken_node` code or stale setup instructions.

**If you see `master` referenced anywhere:**
1. Update it to `main`
2. Ensure `master` is synchronized: `git push origin main:master`

---

## Toolchain: Rust & Cargo

We only use **Cargo** in this project, NEVER any other package manager.

- **Edition:** Rust 2024 (the current tree does not pin a `rust-toolchain.toml`; use a compatible stable toolchain unless a specific task proves otherwise)
- **Dependency versions:** Explicit versions for stability
- **Configuration:** Workspace at root `Cargo.toml`, with primary crate under `crates/franken-node/`
- **Unsafe code:** Forbidden (`#![forbid(unsafe_code)]`)

### Key Dependencies

| Crate | Purpose |
|-------|---------|
| `serde` + `serde_json` | Structured config, reports, bundle payloads, and CLI/API JSON |
| `toml` | Loading `franken_node.toml` and merge-layer configuration data |
| `clap` | CLI argument parsing for `init`, `run`, `migrate`, `verify`, `trust`, `fleet`, `incident`, `bench`, and `doctor` |
| `chrono` | RFC 3339 timestamps |
| `uuid` | Stable identifiers for bundles, operations, incidents, and receipts |
| `sha2` + `hmac` + `hkdf` | Hashing, signing helpers, and derivation material across trust/replay/capability surfaces |
| `ed25519-dalek` + `zeroize` + `serde_cbor` | Key material handling and signed/canonical payload support |
| `base64` + `hex` | Binary/text encoding for signatures, digests, and artifacts |
| `flate2` | Gzip compression for replay bundle chunking/export |
| `tokio` | Shared workspace dependency slot retained for Tokio guardrail/test surfaces; the primary `frankenengine-node` crate is not currently a direct Tokio consumer |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/franken_node](https://github.com/Dicklesworthstone/franken_node) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
