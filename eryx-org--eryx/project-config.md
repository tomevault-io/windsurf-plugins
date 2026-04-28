---
trigger: always_on
description: This document outlines project-specific knowledge for AI agents (and humans) working on this Rust codebase.
---

# AGENTS.md - Eryx Project Guide

This document outlines project-specific knowledge for AI agents (and humans) working on this Rust codebase.

## What is Eryx?

Eryx is a Rust library that executes Python code in a WebAssembly sandbox with async callbacks. It embeds CPython 3.14 compiled to WASM (from [componentize-py](https://github.com/bytecodealliance/componentize-py)).

Key capabilities:
- Run untrusted Python code safely in a sandbox
- Expose Rust async functions as Python callbacks (`await get_time()`)
- Session state persistence for REPL-style usage
- TCP/TLS networking with host-controlled policies
- Pre-compiled WASM for fast startup (~16ms vs ~650ms)

## Project Structure

```
crates/
├── eryx/                  # Main library - public API, Sandbox, callbacks
│   ├── src/
│   │   ├── sandbox.rs     # Sandbox builder and execution
│   │   ├── callback.rs    # TypedCallback, DynamicCallback traits
│   │   ├── session/       # State persistence (InProcessSession)
│   │   └── wasm.rs        # Wasmtime integration
│   ├── examples/          # Usage examples
│   └── benches/           # Criterion benchmarks
├── eryx-runtime/          # WASM runtime packaging
│   ├── runtime.wasm       # Built WASM component (~47MB)
│   ├── runtime.cwasm      # Pre-compiled native code (~52MB)
│   ├── runtime.wit        # WIT interface definition
│   └── libs/              # WASI libraries (zstd compressed)
├── eryx-wasm-runtime/     # Rust code compiled TO WASM (guest side)
│   └── src/lib.rs         # WIT exports, Python FFI
├── eryx-python/           # Python bindings (PyO3/maturin)
├── eryx-precompile/       # CLI tool for AOT compilation
└── eryx-vfs/              # Virtual filesystem for WASM
```

## Tooling

This project uses [mise](https://mise.jdx.dev/) for tooling and task management.

### Quick Start

```bash
mise install           # Install Rust, cargo-nextest
mise run setup         # Build WASM + precompile (one-time)
mise run test          # Run tests with embedded WASM (~0.1s)
mise run ci            # Run all CI checks
```

### Key mise Tasks

```bash
mise run test          # Run tests with embedded WASM (~0.1s)
mise run lint          # cargo clippy with all warnings
mise run lint-fix      # Auto-fix clippy warnings
mise run fmt           # cargo fmt
mise run build-eryx-runtime  # Build Python WASM component
mise run precompile-eryx-runtime-preinit # Pre-compile with preinit snapshot
```

See `mise.toml` for all available tasks.

## WASM Runtime Build Pipeline

Eryx embeds a Python WASM runtime for fast startup. Understanding this pipeline is essential:

### Build Stages

1. **Build WASM** (`mise run build-eryx-runtime`)
   - Compiles `eryx-wasm-runtime` to WebAssembly
   - Outputs: `crates/eryx-runtime/runtime.wasm`

2. **Build Late-Linking Artifacts** (`mise run build-preinit`)
   - Builds native shared libraries for host functions (networking, etc.)
   - Outputs: `liberyx_runtime.so.zst` and related artifacts
   - Required for preinit to link against host capabilities

3. **Precompile with Preinit** (`mise run precompile-eryx-runtime-preinit`)
   - AOT compiles WASM to native code
   - Creates a preinitialized snapshot (Python interpreter already started)
   - Outputs: `crates/eryx-runtime/runtime.cwasm`

4. **Embed at Compile Time**
   - `runtime.cwasm` is included via `include_bytes!()` in Rust code
   - Tests and binaries load this precompiled + preinitialized runtime
   - Result: ~0.1s startup vs ~5s without precompilation

### Why This Matters

- **Without precompilation**: Each test creates a fresh Python runtime (~2-5s in debug)
- **With precompilation**: Tests share the embedded preinitialized runtime (~0.1s total)
- **`SandboxFactory` vs `Sandbox`**: Factory uses the preinit snapshot; if it's stale, behavior differs

## Testing

- **NEVER use `cargo test` directly** - it runs tests sequentially in debug mode and takes minutes
- **Always use `mise run test`** which uses nextest (parallel execution) with embedded/precompiled WASM
- For CI, use `mise run ci` which runs fmt-check, lint, and test

## Common Gotchas

1. **Don't mix workspace and non-workspace dependencies** - always use workspace inheritance
2. **Remember to add `[lints] workspace = true`** to each subcrate's `Cargo.toml`
3. **Use `--workspace` flag** for cargo commands to ensure all crates are covered
4. **Always commit `Cargo.lock`** to version control
5. **Keep lint config in `Cargo.toml`** - avoid separate clippy.toml files

## Build Caching & Staleness Issues

This project has multiple layers of caching that can cause confusing "stale build" issues. Understanding these is critical for debugging.

### Cache Layers

| Cache | Location | Invalidation | When It Gets Stale |
|-------|----------|--------------|-------------------|
| **Cargo target/** | `target/` | Automatic (mtime) | After `git checkout`, cache restore, or clock skew |
| **mise task cache** | Internal | mtime of sources vs outputs | When cargo cache has newer timestamps than sources |
| **Embedded runtime** | `/tmp/eryx-embedded/` | Content hash in filename | Old versions accumulate; shouldn't cause staleness |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eryx-org/eryx](https://github.com/eryx-org/eryx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
