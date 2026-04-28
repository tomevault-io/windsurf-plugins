---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust codebase.
---

# AGENTS.md — frankensearch

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
- **Dependency versions:** Explicit versions for stability
- **Configuration:** Cargo.toml workspace with `workspace = true` pattern
- **Unsafe code:** Forbidden (`#![forbid(unsafe_code)]`)

### Async Runtime: asupersync (MANDATORY — NO TOKIO)

**This project uses [asupersync](/dp/asupersync) exclusively for all async/concurrent operations. Tokio and the entire tokio ecosystem are FORBIDDEN.**

- **Structured concurrency**: `Cx`, `Scope`, `region()` — no orphan tasks
- **Cancel-correct channels**: Two-phase `reserve()/send()` — no data loss on cancellation
- **Sync primitives**: `asupersync::sync::Mutex`, `RwLock`, `OnceCell`, `Pool` — cancel-aware
- **Deterministic testing**: `LabRuntime` with virtual time, DPOR, oracles
- **Native HTTP**: `asupersync::http::h1` for model downloads (replaces reqwest)
- **Rayon is allowed**: For CPU-bound data parallelism (vector dot products). Rayon is not an async runtime.

**Forbidden crates**: `tokio`, `hyper`, `reqwest`, `axum`, `tower` (tokio adapter), `async-std`, `smol`, or any crate that transitively depends on tokio.

**Pattern**: All async functions take `&Cx` as first parameter. The `Cx` flows down from the consumer's runtime — frankensearch does NOT create its own runtime.

### Key Dependencies

| Crate | Purpose |
|-------|---------|
| `asupersync` | Structured async runtime (channels, sync, regions, HTTP, testing) |
| `half` | IEEE 754 f16 float support for quantized vectors |
| `wide` | Portable SIMD (`f32x8`) across x86 SSE2/AVX2 and ARM NEON |
| `memmap2` | Memory-mapped file I/O for zero-copy vector index access |
| `fastembed` | ONNX-based text embeddings (MiniLM-L6-v2 quality tier) |
| `safetensors` + `tokenizers` | Model2Vec static embeddings (potion-128M fast tier) |
| `tantivy` | Full-text BM25 search engine |
| `ort` | ONNX Runtime for cross-encoder reranking |
| `serde` + `serde_json` | Serialization |
| `thiserror` | Ergonomic error type derivation |
| `tracing` | Structured logging and diagnostics |
| `rayon` | Data parallelism for vector search (CPU-bound, not async) |
| `unicode-normalization` | NFC text canonicalization |

### Release Profile

The release build optimizes for performance (this is a library, not a binary):

```toml
[profile.release]
opt-level = 3       # Maximum performance optimization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/frankensearch](https://github.com/Dicklesworthstone/frankensearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
