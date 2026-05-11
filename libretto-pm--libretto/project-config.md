---
trigger: always_on
description: Libretto is a high-performance, Composer-compatible package manager for PHP written in Rust. It aims to be a drop-in replacement for Composer with significantly improved performance through parallel operations, SIMD optimizations, and efficient caching.
---

# CODEX.md - Libretto Project Guide

## Project Overview

Libretto is a high-performance, Composer-compatible package manager for PHP written in Rust. It aims to be a drop-in replacement for Composer with significantly improved performance through parallel operations, SIMD optimizations, and efficient caching.

**Repository**: https://github.com/libretto-pm/libretto  
**License**: MIT OR Apache-2.0  
**MSRV**: Rust 1.75+

## Build & Development Commands

```bash
# Build (debug)
cargo build

# Build (release with optimizations)
cargo build --release
# or use the alias:
cargo br

# Run all tests
cargo test

# Run tests for a specific crate
cargo test -p libretto-core
cargo test -p libretto-resolver

# Run clippy lints (strict - treats warnings as errors)
cargo clippy --all-targets --all-features -- -D warnings

# Check formatting
cargo fmt --all -- --check

# Format code
cargo fmt --all

# Build documentation
cargo doc --no-deps --all-features

# Run benchmarks
cargo bench --package libretto-platform -- --noplot
cargo bench --package libretto-core
cargo bench --package libretto-resolver

# Cross-compilation targets (aliases defined in .cargo/config.toml)
cargo linux-x64    # x86_64-unknown-linux-gnu
cargo linux-arm64  # aarch64-unknown-linux-gnu
cargo macos-x64    # x86_64-apple-darwin
cargo macos-arm64  # aarch64-apple-darwin
cargo windows-x64  # x86_64-pc-windows-msvc
```

## Project Architecture

### Workspace Structure

This is a Cargo workspace with 13 crates under `crates/`:

| Crate | Description |
|-------|-------------|
| `libretto-core` | Core types: Package, Version, Error, JSON utilities, content-addressable hashing |
| `libretto-platform` | Cross-platform compatibility layer: OS detection, SIMD, I/O backends, TLS |
| `libretto-cache` | Multi-tier content-addressable cache with zstd compression and moka |
| `libretto-repository` | Package repository clients (Packagist, private repos) |
| `libretto-resolver` | PubGrub-based dependency resolution (uses astral-pubgrub from uv) |
| `libretto-downloader` | Parallel HTTP/2 package downloading with retry/backoff |
| `libretto-archive` | ZIP/TAR extraction |
| `libretto-vcs` | Git operations for source packages |
| `libretto-autoloader` | PHP autoloader generation with tree-sitter parsing |
| `libretto-plugin-system` | Composer plugin compatibility |
| `libretto-audit` | Security vulnerability checking |
| `libretto-lockfile` | Atomic composer.lock file management |
| `libretto-cli` | CLI binary and command implementations |

### Dependency Flow

```
libretto-cli
├── libretto-core (foundation)
├── libretto-platform (OS abstractions)
├── libretto-cache
├── libretto-repository
├── libretto-resolver
├── libretto-downloader
│   ├── libretto-archive
│   └── libretto-vcs
├── libretto-autoloader
├── libretto-plugin-system
└── libretto-audit
```

### CLI Commands

The main binary is `libretto` (defined in `libretto-cli`):

- `install` - Install dependencies from composer.json/composer.lock
- `update` - Update dependencies to latest versions
- `require` - Add a package to dependencies
- `remove` - Remove a package from dependencies
- `search` - Search for packages
- `show` - Show package information
- `init` - Initialize a new composer.json
- `validate` - Validate composer.json
- `dump-autoload` - Regenerate autoloader
- `audit` - Check for security vulnerabilities
- `cache:clear` - Clear the package cache

## Code Style & Conventions

### Rust Conventions

- **Edition**: 2024
- **Unsafe code**: Denied (`#![deny(unsafe_code)]` in most crates)
- **Lints**: Strict clippy configuration (pedantic, nursery, cargo warnings)
- **Unwrap**: `clippy::unwrap_used = "warn"` - prefer `?` or explicit error handling

### File Header Pattern

Each crate's lib.rs follows this pattern:
```rust
//! Crate description and documentation.

#![deny(clippy::all)]
#![allow(clippy::module_name_repetitions)]
```

### Error Handling

- Use `thiserror` for error type definitions
- Use `anyhow` for CLI error propagation
- Custom `Error` enum in `libretto-core` for domain errors
- Each crate may have its own error types that convert to core errors

### Serialization

- Primary JSON: `sonic-rs` (high-performance SIMD JSON)
- Serde integration via `serde` with derive
- Binary caching: `rkyv` for zero-copy deserialization

### Concurrency Patterns

- `tokio` for async runtime
- `rayon` for CPU-bound parallelism
- `dashmap` for concurrent hash maps
- `parking_lot` for synchronization primitives
- `moka` for concurrent caching with TTL

### Global Allocator

`libretto-core` sets mimalloc as the global allocator:
```rust
#[global_allocator]
static GLOBAL: mimalloc::MiMalloc = mimalloc::MiMalloc;
```

## Key Technical Details

### Version Resolution

The resolver uses `astral-pubgrub` (the PubGrub implementation from the `uv` project) for dependency resolution. It supports:
- All Composer constraint formats (^, ~, *, ranges, OR)
- Stability flags (@dev, @alpha, @beta, @RC, @stable)
- Dev branches (dev-* prefixes)
- Replace and provide declarations

### Platform Support

Full support for:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libretto-pm/libretto](https://github.com/libretto-pm/libretto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
