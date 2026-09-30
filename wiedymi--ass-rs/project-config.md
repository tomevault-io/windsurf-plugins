---
trigger: always_on
description: This file provides guidance to coding agents working in this repository.
---

# AGENTS.md

This file provides guidance to coding agents working in this repository.
`CLAUDE.md` is a symlink to this file, so tools looking for either name read the
same instructions.

> **Status (2026-06):** Active but early. `ass-core` and `ass-editor` are
> feature-complete and CI-backed; `ass-renderer` is a work in progress — its
> software backend works, GPU backends are experimental. See
> [Architecture Overview](#architecture-overview) for per-crate state.

## Build/Test/Lint Commands
- Build (workspace): `cargo build` — builds all three crates with default features.
- Build single crate: `cargo build -p ass-core` (or `-p ass-editor`, `-p ass-renderer`).
- Build with extras: `cargo build [--release] [--features="simd,arena"]`
- Test single: `cargo test test_name`
- Test file: `cargo test --test file_name`
- Format: `cargo fmt --all` (gate: `cargo fmt --all -- --check`)
- Benchmarks: `cargo bench --features="benches"`
- WASM tests: `wasm-pack test --chrome`
- Fuzzing: `cargo +nightly fuzz run tokenizer`

> **IMPORTANT — do not use `--all-features` on this workspace.** `ass-renderer`
> is std-only (it links `fontdb`, `tiny-skia`, `rustybuzz`, `rayon`), so its
> `nostd` feature is mutually exclusive with `software-backend`/`web-backend`.
> `--all-features` turns both on at once and fails to compile — this is by
> design, not a regression. Use the scoped feature matrix below (it mirrors
> `.github/workflows/ci.yml`), which is the real supported surface.

### Test (full supported matrix)
The renderer is exercised by the `full`/`full,simd-full` `--workspace` combos;
the `no_std`-flavored combos are scoped to the `no_std`-capable crates
(`ass-core`, `ass-editor`). Each combo also runs under `--release` in CI.
```bash
# Whole workspace (includes ass-renderer):
cargo test --workspace --no-default-features --features full
cargo test --workspace --no-default-features --features full,simd-full
# no_std-capable crates only:
cargo test -p ass-core -p ass-editor --no-default-features --features minimal
cargo test -p ass-core -p ass-editor --no-default-features --features minimal,nostd
```

### Lint (full supported matrix)
Same scoping as Test; run each with and without `--all-targets`.
```bash
cargo clippy --workspace --no-default-features --features full -- -D warnings
cargo clippy --workspace --all-targets --no-default-features --features full -- -D warnings
cargo clippy --workspace --all-targets --no-default-features --features full,simd-full -- -D warnings
cargo clippy -p ass-core -p ass-editor --all-targets --no-default-features --features minimal -- -D warnings
cargo clippy -p ass-core -p ass-editor --all-targets --no-default-features --features minimal,nostd -- -D warnings
```

### Other CI gates
```bash
# no_std build (core + editor only):
cargo build -p ass-core --no-default-features --features minimal,nostd
cargo build -p ass-editor --no-default-features --features minimal,nostd
# Docs (warnings are errors) — curated, working feature surface:
RUSTDOCFLAGS="-D warnings" cargo doc --workspace --no-deps --no-default-features --features full,analysis,plugins,simd,serde,unicode-wrap
# MSRV (1.82):
cargo check --workspace --no-default-features --features full,analysis,plugins,simd,serde,unicode-wrap
# Coverage (scope to a working combo, not --all-features):
cargo tarpaulin --workspace --no-default-features --features full
```

> **Renderer note:** `ass-renderer`'s `libass-compare` feature is a stub
> (the `libass` crate dependency is commented out and its FFI shim in
> `src/debug/libass_ffi.rs` does not currently compile). It is intentionally
> excluded from `default`/`full`. Do not re-add it to those sets until the
> libass integration is restored and builds clean.

## Code Style Guidelines
- **Safety**: No unsafe code allowed - absolutely forbidden
- **Imports**: Group by scope (std → external → internal); specific imports for frequent items
- **Formatting**: 4-space indentation; standard Rust formatting; rustfmt defaults
- **Types**: Use zero-copy spans (`&str`) for performance; custom Result types; feature-gated optimizations
- **Naming**: CamelCase for types; snake_case for functions/files; SCREAMING_SNAKE_CASE for constants
- **Error Handling**: Use `thiserror` for enums (no `anyhow`); prefer Result over panics
- **Documentation**: Use only Rustdoc (`///` for public, `//!` for modules); no inline comments; example-heavy
- **Testing**: >90% test coverage required; fuzz hot paths; WASM tests mandatory
- **Dependencies**: Minimal deps (<50KB); pin versions in workspace; feature-gate heavy ones
- **Performance**: <5ms/operation, <1.1x input memory; use zero-copy, arenas, SIMD where beneficial
- **Modularity**: Submodules per concern; traits for extensibility; file size <200 LOC
- **Features**: Consistent across crates; default minimal; gate extras; maintain no_std compatibility
- **Workarounds**: Never use workarounds, bypass, or skip logic; never use `allow(clippy)` to fix something

## Architecture Overview

### Project Structure
ASS-RS is a modular ASS (Advanced SubStation Alpha) subtitle toolkit aiming to
surpass libass in performance and safety. The workspace currently contains three
crates:

- **`crates/ass-core`** (v0.1.1) — **Complete.** Zero-copy parser, tokenizer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wiedymi/ass-rs](https://github.com/wiedymi/ass-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
