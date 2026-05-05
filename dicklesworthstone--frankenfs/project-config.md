---
trigger: always_on
description: Provides a unified filesystem that can mount ext4/btrfs images in Linux userspace (FUSE), with MVCC/COW transaction semantics for concurrent writers and fountain-code-based self-healing repair for durability.
---

# AGENTS.md — FrankenFS (ffs)

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

- **Edition:** Rust 2024 (nightly required — see `rust-toolchain.toml`)
- **Dependency versions:** Explicit versions for stability and reproducibility
- **Configuration:** Cargo.toml workspace with `workspace = true` pattern
- **Unsafe code:** Forbidden (`#![forbid(unsafe_code)]` at crate roots + workspace lint)

### Async Runtime: asupersync (MANDATORY — NO TOKIO)

**This project uses [asupersync](/dp/asupersync) exclusively for all async/concurrent operations. Tokio and the entire tokio ecosystem are FORBIDDEN.**

- **Structured concurrency**: `Cx`, `Scope`, `region()` — no orphan tasks
- **Cancel-correct channels**: Two-phase `reserve()/send()` — no data loss on cancellation
- **Sync primitives**: `asupersync::sync::Mutex`, `RwLock`, `OnceCell`, `Pool` — cancel-aware
- **Deterministic testing**: `LabRuntime` with virtual time, DPOR, oracles
- **No ambient authority.** Cancellation/deadline-sensitive operations should take `&asupersync::Cx`

**Forbidden crates**: `tokio`, `hyper`, `reqwest`, `axum`, `tower` (tokio adapter), `async-std`, `smol`, or any crate that transitively depends on tokio.

**Pattern**: All async functions take `&Cx` as first parameter. The `Cx` flows down from the consumer's runtime — FrankenFS does NOT create its own runtime.

### Key Dependencies

| Crate | Purpose |
|-------|---------|
| `asupersync` | Structured async runtime (channels, sync, regions, `Cx`, deterministic lab runtime, RaptorQ pipeline) |
| `ftui` (frankentui) | Terminal UX for CLI diagnostics and tooling |
| `bitflags` | On-disk flag field modeling |
| `blake3` | Fast cryptographic hashing for block integrity |
| `crc32c` | CRC32C checksums (ext4/btrfs on-disk format) |
| `xxhash-rust` | XXH3 non-cryptographic hashing for hot paths |
| `memchr` | Hot path byte scanning |
| `smallvec` | Hot path stack-allocated vectors |
| `fuser` | FUSE userspace filesystem interface |
| `serde` + `serde_json` | Fixtures, conformance vectors, metadata reports |
| `thiserror` | Ergonomic error type derivation |
| `tracing` | Structured logging and diagnostics |
| `criterion` | Performance benchmarks |
| `proptest` | Property-based testing |
| `tempfile` | Temporary file/directory creation for tests |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/frankenfs](https://github.com/Dicklesworthstone/frankenfs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
