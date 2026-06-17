---
trigger: always_on
description: Audience: agentic coding assistants and maintainers working in this repo.
---

# AGENTS.md — rust-openzl (vendored OpenZL)

Audience: agentic coding assistants and maintainers working in this repo.
Scope: the entire `rust-openzl` workspace and its sub-crates.

Goal: deliver robust, safe, and ergonomic Rust bindings for OpenZL using a vendored source tree at `openzl-sys/vendor/openzl` (vendored for crates.io), with a clear roadmap from MVP to advanced features.

**Status**: MVP complete and published to crates.io! 🎉
- rust-openzl-sys v0.1.0: https://crates.io/crates/rust-openzl-sys
- rust-openzl v0.1.0: https://crates.io/crates/rust-openzl

## Quick Start (Using Published Crate)

Add to your `Cargo.toml`:
```toml
[dependencies]
rust-openzl = "0.1.0"
```

Example usage:
```rust
use rust_openzl::{compress_numeric, decompress_numeric};

let data: Vec<u32> = (0..10000).collect();
let compressed = compress_numeric(&data)?;
let decompressed: Vec<u32> = decompress_numeric(&compressed)?;
assert_eq!(data, decompressed);
```

See [examples/](openzl/examples/) for more usage patterns.

## Ground Rules
- Always build against the vendored OpenZL in `openzl-sys/vendor/openzl`. Do not depend on system installations.
- Do not modify upstream OpenZL sources unless absolutely required. Prefer thin shims in `openzl-sys/src/` or upstream PRs.
- Keep `rust-openzl-sys` unsafe and minimal; put ergonomic, safe APIs in `rust-openzl`.
- Preserve reproducibility: vendored sources are committed directly (no git submodules for crates.io compatibility).
- Avoid adding unrelated dependencies. Prefer std + well-known crates where necessary.

## Repository Layout
- `openzl-sys/vendor/openzl/` — vendored OpenZL C/C++ sources and CMake build (includes zstd as nested dependency).
- `openzl-sys/` — unsafe FFI crate (`rust-openzl-sys`): CMake build, C shim helpers, bindgen-generated Rust declarations.
- `openzl/` — safe wrapper crate (`rust-openzl`): RAII, Result-based error handling, type-safe and ergonomic API.
- `Cargo.toml` (workspace), `.gitignore`, `README.md`, `AGENTS.md`, `CHANGELOG.md`.

## Prerequisites (Local + CI)
- Build tools: `cmake >= 3.20`, C toolchain (clang or gcc), `make`/`ninja`.
- Bindgen: `clang` and `libclang` available on PATH (bindgen 0.69).
- Vendored sources are included in the repository (no git submodule setup needed).

Quick bootstrap:
```bash
cargo build -p rust-openzl
cargo test -p rust-openzl
```

Or use the published crate:
```bash
cargo add rust-openzl
```

## OpenZL Architecture Understanding

**CRITICAL**: OpenZL is fundamentally a **graph-based typed compression** library, NOT a generic compressor like zlib/zstd.

- **Compression Graphs** are the CORE concept: they define HOW to compress specific data structures
- **Serial compression** (`compress_serial`/`decompress_serial`) is just a simple compatibility shim
- **TypedRef compression** REQUIRES compression graphs - you cannot compress typed data without specifying which algorithms to use
- Standard graphs available: `ZL_GRAPH_ZSTD`, `ZL_GRAPH_NUMERIC`, `ZL_GRAPH_FIELD_LZ`, `ZL_GRAPH_STORE`, etc.

See `vendor/openzl/examples/zs2_struct.c` for canonical usage pattern.

## Long, Detailed Sequence of Next Steps

The following steps are ordered for incremental value and easier review. Treat each numbered section as a checkpoint with a commit.

### 1) ✅ Stabilize the current build and baseline
- Verify `cargo build -p openzl` succeeds on Linux and macOS.
- Validate static linking: inspect the build dir (`target/debug/build/openzl-sys-*/out`) for `libopenzl.a` and link flags.
- Confirm that zstd is linked statically (presence of `libzstd.a`).
- Add minimal smoke tests in `openzl/tests/smoke.rs` to exercise `compress_serial`/`decompress_serial` round-trip.

### 2) ✅ Harden bindgen configuration
- Ensure bindgen allows only `ZL_*` symbols via allowlist.
- Use `--default-enum-style rust` for type-safe enums.
- Avoid Hash/Ord derives for structs with function pointers (causes warnings).
- Add layout tests for ABI verification.
- Document binding regeneration process in README.

### 3) ✅ Error handling and diagnostics
- C shim helpers for:
  - `openzl_error_code_to_string(ZL_ErrorCode)` using `ZL_ErrorCode_toString`.
  - Accessors for warnings arrays (`ZL_CCtx_getWarnings`, `ZL_DCtx_getWarnings`)
  - Error code/name extraction from `ZL_Error`
- In `openzl`, add `Warning` type with `warnings()` methods on contexts.
- Ensure all `ZL_Report` values are checked via shims; never inspect unions directly in Rust.

### 4) ✅ Compression graphs (CORE FUNCTIONALITY)

**This is the heart of OpenZL** - must come before TypedRef compression APIs.

Phase A: Basic graph infrastructure
- Wrap `ZL_Compressor`:
  - `Compressor::new()` -> `ZL_Compressor_create()`
  - `Drop` -> `ZL_Compressor_free()`
  - `set_parameter()` for global parameters
  - `warnings()` for graph validation errors
- Expose `ZL_GraphID` as opaque type with validity checking (`ZL_GraphID_isValid`).
- Expose standard graph constants as Rust constants:
  - `GRAPH_ZSTD` = `ZL_StandardGraphID_zstd`
  - `GRAPH_NUMERIC` = `ZL_StandardGraphID_select_numeric`
  - `GRAPH_STORE` = `ZL_StandardGraphID_store`
  - `GRAPH_FIELD_LZ` = `ZL_StandardGraphID_field_lz`
  - etc.

Phase B: Graph function API (stateless compression)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vitorpy/rust-openzl](https://github.com/vitorpy/rust-openzl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
