---
trigger: always_on
description: This file provides guidance for agentic coding agents working on this codebase.
---

# AGENTS.md

This file provides guidance for agentic coding agents working on this codebase.

## Build, Lint, and Test Commands

### Core Commands
- `cargo build` - Compile the project (development profile)
- `cargo build --release` - Compile optimized release build
- `cargo check` - Check for errors without building (faster)
- `cargo test` - Run all tests
- `cargo test --lib` - Run library tests only
- `cargo test --test <test_name>` - Run specific integration test
- `cargo test <test_name>` - Run a single test function by name
- `cargo clippy` - Run linter to catch common mistakes
- `cargo clippy -- -D warnings` - Fail on clippy warnings
- `cargo fmt` - Format all code using rustfmt
- `cargo fmt --check` - Check formatting without modifying files

### Protobuf Regeneration
- The `build.rs` script generates Rust code from `.proto` files
- Run `cargo build` to regenerate protobuf code if `.proto` files change
- Proto file location: `src/raft/proto/raft.proto`

### Code Quality Requirements

**All code changes MUST pass the following checks before submission:**

#### Formatting Check
```bash
# Check code formatting
cargo fmt --all -- --check

# Auto-fix formatting issues
cargo fmt --all
```

#### Clippy Check
```bash
# Run clippy (treat warnings as errors)
cargo clippy --all-features -- -D warnings
```

**⚠️ IMPORTANT: After every code change, ensure both commands pass with ZERO errors and ZERO warnings!**

## Code Style Guidelines

### Type Import Rules

**DO NOT** use long path references to types directly in code:

```rust
// ❌ Wrong
pub fn to_openai_tool(&self) -> async_openai::types::chat::ChatCompletionTools {
    // ...
}

// ❌ Wrong
pub fn process_response(response: async_openai::types::chat::CreateChatCompletionResponse) {
    // ...
}
```

**MUST** use `use` to import types at the top of the file, then use short names:

```rust
// ✅ Correct
use async_openai::types::chat::ChatCompletionTools;

pub fn to_openai_tool(&self) -> ChatCompletionTools {
    // ...
}

// ✅ Correct
use async_openai::types::chat::CreateChatCompletionResponse;

pub fn process_response(response: CreateChatCompletionResponse) {
    // ...
}
```

### Formatting
- **Indentation**: 2 spaces (configured in `rustfmt.toml`)
- **Import ordering**: Automatic reordering enabled (`reorder_imports = true`)
- **Edition**: Rust 2024
- **Run `cargo fmt` before committing changes**

### Module Organization
- Library root (`src/lib.rs`): Declare public modules with `pub mod`
- Module directories: Use `mod.rs` for module contents
- Re-exports: Use `pub use` in `mod.rs` to expose types from submodules
- Example: `src/raft/types/mod.rs` re-exports all types from submodules

### Imports
- Group external std/core imports first, then third-party, then local crate imports
- Use `use crate::` for local crate imports within the project
- Prefer absolute paths over `super::` for clarity

### Types and Serialization
- Use `serde::{Serialize, Deserialize}` derive macros for types that need serialization
- Use `#[serde(default = "function_name")]` for default values in config
- Custom types are preferred over raw primitives (e.g., `NodeId` instead of `u64`)
- Use `anyerror::AnyError` for generic error wrapping
- Use `thiserror::Error` for custom error enums with context

### Error Handling
- Define custom error enum in `src/error/mod.rs` using `thiserror::Error`
- Use `#[from]` attribute for automatic error conversion
- Create `Result<T>` type alias: `pub type Result<T> = std::result::Result<T, RockRaftError>`
- Error variants should be descriptive with error messages
- Use `map_err()` to add context to errors: `.map_err(|e| AnyError::error(format!("context: {}", e)))`

### Naming Conventions
- **Types/Structs/Enums**: PascalCase (`RaftNode`, `RocksdbConfig`)
- **Functions/Methods**: snake_case (`create`, `is_in_cluster`)
- **Constants**: UPPER_SNAKE_CASE (`LOG_META_FAMILY`)
- **Private fields**: snake_case
- **Acronyms in names**: Keep uppercase (`RocksDB`, `Raft`, `KV`)

### Async Concurrency
- Use `tokio::task::spawn` for concurrent tasks
- Use `tokio::sync::broadcast` for shutdown signals
- Use `Arc<T>` for shared ownership across threads
- Use `std::sync::Mutex` or `tokio::sync::Mutex` for synchronization
- Prefer `tokio::time::timeout` for operations with deadlines

### Testing Patterns
- Place tests in `#[cfg(test)]` modules at the bottom of files
- Use `#[tokio::test]` for async test functions
- Return `Result<()>` from tests to use `?` operator
- Use `tempfile` crate for temporary directories in tests
- Test function naming: `test_<feature>_<scenario>`
- Helper functions: Create private `create_test_*` helpers for setup
- For cluster tests that stop/restart the cluster, ensure the cluster is restarted at the end for subsequent tests

### Comments and Documentation
- Use `///` for public item documentation (rustdoc)
- Use `//!` for module-level documentation
- Keep comments concise and focused on "why" not "what"
- No inline comments unless explaining complex logic

### Raft-Specific Patterns
- Raft nodes are created via `RaftNodeBuilder::build(&config)`
- Use `assume_leader()` to get a `LeaderHandler` for write operations
- Handle `ForwardToLeader` errors to redirect to current leader

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lichuang/rockraft](https://github.com/lichuang/rockraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
