---
trigger: always_on
description: Phantom Fragment is a high-performance container alternative using Linux kernel features (namespaces, seccomp, Landlock, cgroups). Codebase: Primarily Rust with Zig for low-level components (zygote pool, memory management).
---

# AGENTS.md - Phantom Fragment Development Guide

## Project Overview

Phantom Fragment is a high-performance container alternative using Linux kernel features (namespaces, seccomp, Landlock, cgroups). Codebase: Primarily Rust with Zig for low-level components (zygote pool, memory management).

## Build Commands

```bash
# Build all components
cargo build --release
cargo build --workspace

# Build specific package
cargo build --package phantom-cli
cargo build --package execution-rs

# Check without building
cargo check --workspace

# Build Zig components
cd src/memory/discipline/memory-zig && zig build
cd src/core/zygote-rs && zig build
```

## Test Commands

```bash
# Run all tests
cargo test --workspace

# Run tests for specific package
cargo test --package execution-rs
cargo test --package landlock-rs

# Run single test by name
cargo test --package execution-rs test_mode_selection
cargo test test_capability_detection

# Run tests with output
cargo test --workspace -- --nocapture

# Run ignored tests
cargo test -- --ignored

# Run integration tests
cargo test --package integration-tests
```

## Lint & Format Commands

```bash
# Format Rust code
cargo fmt

# Format Zig code
zig fmt src/memory/discipline/memory-zig/src/main.zig
zig fmt src/core/zygote-rs/src/zygote.zig

# Run Clippy
cargo clippy --workspace --all-targets -- -D warnings

# Check without building
cargo check --workspace
```

## Code Style Guidelines

### Rust

**Imports:** Group by external crates first, then internal modules.

```rust
use anyhow::Result;
use serde::{Deserialize, Serialize};
use std::path::PathBuf;

use crate::executor::BuildContext;
use types_rs::PhantomError;
```

**Naming:**
- Types/Structs: `PascalCase` (`AdaptiveEngine`, `SecurityPolicy`)
- Functions/Methods: `snake_case` (`select_mode`, `apply_seccomp`)
- Constants: `SCREAMING_SNAKE_CASE` (`PHANTOM_ERROR_SUCCESS`)
- Modules: `snake_case` (`bpf_loader`, `task_analyzer`)
- Crates: `kebab-case` (`execution-rs`, `image-puller`)

**Error Handling:** Use `anyhow::Result` for application code, `thiserror` for custom errors.

```rust
#[derive(Error, Debug)]
pub enum BpfError {
    #[error("BPF-LSM not supported")]
    NotSupported,
    #[error("Failed to load policy: {0}")]
    LoadPolicy(String),
}
```

**Structs:** Use `#[derive]` attributes. Implement `Default` where sensible.

```rust
#[derive(Debug, Clone, Default)]
pub struct RiskProfile {
    pub network_access: bool,
    pub file_write: bool,
    pub privileged_ops: bool,
    pub untrusted_source: bool,
}
```

**Documentation:** Use `//!` for crate-level docs, `///` for public items.

```rust
/// Adaptive Execution Engine
/// 
/// Automatically selects the optimal execution mode based on workload risk.
pub struct AdaptiveEngine { ... }
```

**Tests:** Place unit tests in same file with `#[cfg(test)]` module.

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_capability_detection() {
        let loader = BpfProgramLoader::new();
        assert!(loader.is_ok());
    }
}
```

### Zig

**Exports:** Use `export fn` for C-compatible FFI with `c_int` returns.

```zig
export fn phantom_zygote_fork() c_int {
    // ...
}
```

**Naming:** `snake_case` for functions, `PascalCase` for types.

**Formatting:** Use `zig fmt`.

## Workspace Structure

```
src/
├── cli/phantom-cli/          # Main CLI binary
│   └── src/
│       ├── main.rs           # CLI entry point (~205 lines, command dispatch)
│       ├── commands/         # Modular command pattern (~25 modules)
│       │   ├── mod.rs
│       │   ├── run.rs        # Run command
│       │   ├── create.rs     # Create command
│       │   ├── list.rs       # List command
│       │   ├── logs.rs       # Logs command
│       │   ├── delete.rs     # Destroy command
│       │   ├── health.rs     # Health check
│       │   ├── metrics.rs    # Metrics
│       │   ├── warm.rs       # Zygote warm-up
│       │   └── ...           # Other subcommands
│       ├── config.rs         # Config + PhantomPaths
│       ├── ui.rs             # Standardized UI output
│       ├── io_utils.rs       # Shared file I/O utilities
│       └── fragment_registry.rs
├── core/
│   ├── execution/execution-rs    # Adaptive execution engine
│   ├── types/types-rs            # Shared FFI-safe types
│   ├── zygote-rs                 # Zygote pool (Zig)
│   ├── task-analyzer-rs          # Command analysis
│   ├── network-rs                # Network management
│   ├── compression-rs            # Compression utilities
│   ├── wasm-rs                   # WebAssembly execution
│   └── config-rs                 # Configuration handling
├── security/
│   ├── bpf-lsm/bpf-lsm-rs        # BPF-LSM integration
│   ├── orchestration/security-rs # Security manager
│   ├── policy-dsl-rs             # Policy DSL
│   ├── seccomp-rs                # Syscall filtering
│   ├── landlock-rs               # Filesystem sandboxing
│   ├── cgroups-rs                # Resource limits
│   └── capabilities-rs           # Linux capabilities
├── memory/
│   └── discipline/
│       ├── memory-rs             # Memory management (Rust)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Intro0siddiqui/Phantom-Fragment](https://github.com/Intro0siddiqui/Phantom-Fragment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
