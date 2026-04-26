---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust codebase.
---

# AGENTS.md — xf (X Archive Finder)

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
- **Configuration:** Cargo.toml only (single-crate project, no workspace)
- **Unsafe code:** Forbidden (`#![forbid(unsafe_code)]` via crate lints)

### Async Runtime: Tokio

This project uses **Tokio** for async operations.

- `tokio` with features: `rt-multi-thread`, `macros`, `fs`, `io-util`, `time`, `signal`
- Background daemon uses Tokio channels and tasks for client/server communication

### Key Dependencies

| Crate | Purpose |
|-------|---------|
| `tantivy` | Full-text BM25 search engine and query parser |
| `rusqlite` | SQLite storage for metadata, stats, and FTS5 fallback |
| `serde` + `serde_json` | Archive parsing and output serialization |
| `rmp-serde` | MessagePack serialization for daemon protocol |
| `toon_rust` | Serialization utilities |
| `clap` + `clap_complete` | CLI parsing and shell completions |
| `rayon` | Parallel parsing of archive files |
| `chrono` + `chrono-english` | Timestamp parsing, formatting, and natural-language date input |
| `tracing` + `tracing-subscriber` | Structured logging with env-filter |
| `fastembed` | ONNX-based text embeddings (MiniLM-L6-v2 quality tier) |
| `safetensors` + `tokenizers` | Model2Vec static embeddings (potion fast tier) |
| `ort` | ONNX Runtime for static embedding models and cross-encoder reranking |
| `half` | IEEE 754 f16 float support for quantized vector storage |
| `wide` | Portable SIMD (`f32x8`) for fast dot products |
| `ring` | SHA256 content hashing |
| `rich_rust` | Rich terminal output (optional feature) |
| `colored` + `indicatif` + `console` | Terminal output and progress bars |
| `rustyline` | REPL line editing |
| `unicode-normalization` | NFC text canonicalization |
| `thiserror` | Ergonomic error type derivation |

### Release Profile

The release build optimizes for binary size:

```toml
[profile.release]
opt-level = "z"     # Optimize for size (lean binary for distribution)
lto = true          # Link-time optimization
codegen-units = 1   # Single codegen unit for better optimization
panic = "abort"     # Smaller binary, no unwinding overhead
strip = true        # Remove debug symbols
```

---

## Code Editing Discipline

### No Script-Based Changes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/xf](https://github.com/Dicklesworthstone/xf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
