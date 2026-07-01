---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
cargo build                            # Build the compiler (no LLVM backend)
cargo test                             # Run non-codegen tests (lexer, parser, resolver, typechecker, effect, ownership, interpreter)
cargo test --features llvm             # Run ALL tests including codegen E2E + memory_sanitizer (ASAN)
cargo test lexer                       # Run a single test file (e.g., tests/lexer.rs)
cargo test -- test_name                # Run a single test by name
cargo clippy --all --all-targets -- -D warnings  # Lint (must be clean before declaring work done)
cargo fmt --all                        # Format all files
cargo fmt --all -- --check             # Verify formatted (must be clean before declaring work done — peer to clippy)
```

**`cargo fmt --all -- --check` is a hard pre-commit gate, peer to clippy.** Both must clear before any commit lands. **First action of any new coding session or slice:** run `cargo fmt --all -- --check`. If it fails, fix with `cargo fmt --all` and land as a standalone `chore: cargo fmt cleanup` commit *before* starting feature work. Don't pull fmt drift into a feature commit; don't surgically revert drift to keep a commit scoped — both patterns push cleanup to CI and let drift accumulate in the meantime.

**Use `--all-targets`, not `--tests`, on the clippy gate.** `--tests` only builds the test target (cfg(test)), so any lint that fires only in production cfg slips through. The runtime crate has cfg-gated type definitions (e.g. `KARAC_SPAWN_SITES` is `extern KaracSpawnSiteEntry` in production but a `SpawnSiteEntryStandIn` wrapper under cfg(test)) — clippy lints on those code paths only fire in the cfg where they're real, and CI runs `cargo clippy --all -- -D warnings` (no `--tests`). `--all-targets` builds lib + bins + tests + examples + benches, each in its own cfg, so it covers both surfaces.

**Codegen and memory-sanitizer tests are gated on `--features llvm`.** Plain `cargo test` will skip `tests/codegen.rs`, `tests/par_codegen.rs`, and `tests/memory_sanitizer.rs` entirely (the modules are `#[cfg(feature = "llvm")]`). Always use `--features llvm` when verifying codegen-related work; otherwise you will miss real regressions.

**Codegen E2E + memory_sanitizer require the runtime library.** One-time setup on a fresh checkout:

```bash
# Lean archive first (rustls-free, native net kept) — built into the canonical name, then renamed.
cargo rustc -p karac-runtime --release --no-default-features --features net --crate-type staticlib
cp target/release/libkarac_runtime.a target/release/libkarac_runtime_min.a
# Full archive (TLS on) overwrites the canonical name — must run SECOND.
cargo rustc -p karac-runtime --release --crate-type staticlib   # target/release/libkarac_runtime.a
# WASM archive (phase-10 `--target=wasm_wasi`) — separate target dir, no clobber risk.
cargo rustc -p karac-runtime --release --target wasm32-wasip1 --no-default-features --crate-type staticlib
cp target/wasm32-wasip1/release/libkarac_runtime.a target/release/libkarac_runtime_wasm.a
# Threaded WASM archive (phase-10 `--features wasm-threads`) — separate target dir too.
# Prereq: `rustup target add wasm32-wasip1-threads` (its sysroot is the only one whose
# wasi-libc is built with atomics — required for the --shared-memory link).
cargo rustc -p karac-runtime --release --target wasm32-wasip1-threads --no-default-features --features wasm-threads --crate-type staticlib
cp target/wasm32-wasip1-threads/release/libkarac_runtime.a target/release/libkarac_runtime_wasm_threads.a
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [karalang/kara](https://github.com/karalang/kara) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
