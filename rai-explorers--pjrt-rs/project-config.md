---
trigger: always_on
description: This file provides guidance for AI agents working in the pjrt-rs repository.
---

# AGENTS.md - Coding Guidelines for pjrt-rs

This file provides guidance for AI agents working in the pjrt-rs repository.

## Build, Lint, and Test Commands

### Building
```bash
# Build the entire workspace
cargo build --verbose

# Build a specific package
cargo build -p pjrt
cargo build -p pjrt-sys

# Build for release
cargo build --release
```

### Code Formatting
```bash
# Format all code (requires nightly toolchain)
rustup component add rustfmt --toolchain nightly
cargo +nightly fmt --all

# Check formatting without modifying
cargo +nightly fmt --all -- --check
```

### Linting
```bash
# Run Clippy on workspace with tests and examples
cargo clippy --workspace --tests --examples -- -D warnings

# Install required components
rustup component add clippy rustfmt
```

### Testing
```bash
# Run all tests
cargo test

# Run tests for a specific package
cargo test --all-features -p pjrt
cargo test --all-features -p pjrt-sys

# Run a single test by name
cargo test --all-features test_error_code_values
cargo test --all-features -p pjrt test_client_debug_impl

# Run tests with output
cargo test -- --nocapture --all-features

# Note: Integration tests and examples require PJRT_PLUGIN_PATH to be set
cargo test -- --nocapture --all-features 2>&1 | head -20  # See which tests run/fail

# Run tests in release mode (faster for heavy tests)
cargo test --release
```

### Prerequisites
- Install protoc: Required for building (uses prost for protobuf)
- Rust nightly toolchain: Required for rustfmt unstable features
- PJRT Plugin: Required for integration tests and examples, assume it is configurated by user.
  - Set `PJRT_PLUGIN_PATH` environment variable to the path of the PJRT plugin (e.g., `pjrt_c_api_cpu_plugin.so`)
  - Example: `export PJRT_PLUGIN_PATH=/path/to/pjrt_c_api_cpu_plugin.so`
- .env defined required environment required to run the examples

## Code Style Guidelines

### Rust Edition and Toolchain
- **Edition**: 2021
- **Toolchain**: Nightly (required for rustfmt unstable features)

### Formatting (rustfmt.toml)
```toml
newline_style = "Unix"
use_field_init_shorthand = true
style_edition = "2021"
imports_granularity = "Module"
group_imports = "StdExternalCrate"
format_code_in_doc_comments = true
format_macro_bodies = true
format_macro_matchers = true
```

### Import Organization
Imports must be grouped in this order (enforced by rustfmt):
1. **Standard library**: `use std::...`
2. **External crates**: `use bon::...`, `use pjrt_sys::...`
3. **Internal (crate)**: `use crate::...`

Example:
```rust
use std::borrow::Cow;
use std::ffi::c_void;
use std::rc::Rc;
use std::slice;

use bon::bon;
use pjrt_sys::{PJRT_Client, PJRT_Error_Code};

use crate::{Api, Buffer, Device, Result};
```

### Naming Conventions
- **Types**: PascalCase (e.g., `Client`, `HostBuffer`, `LoadedExecutable`)
- **Functions/Methods**: snake_case (e.g., `create_client`, `platform_name`)
- **Constants**: SCREAMING_SNAKE_CASE for true constants
- **Modules**: snake_case (e.g., `mod host_buffer;`)
- **Generic parameters**: Single uppercase letters or descriptive names (e.g., `T`, `Item`)

### Error Handling
- Use `thiserror` derive macro for error types
- Define a crate-wide `Result<T>` type alias
- Use `?` operator for error propagation
- Provide detailed error messages with context

Example:
```rust
#[derive(thiserror::Error, Debug)]
pub enum Error {
    #[error("pjrt error {msg}\n{backtrace}")]
    PjrtError { msg: String, code: ErrorCode, backtrace: String },
    #[error("invalid argument: {0}")]
    InvalidArgument(String),
    #[error("io error: {0}")]
    IoError(#[from] std::io::Error),
}

pub type Result<T> = std::result::Result<T, Error>;
```

### Types and Type Safety
- Prefer strong types over primitive types (e.g., `GlobalDeviceId`, `LocalHardwareId`)
- Use `#[repr(i32)]` for C-compatible enums
- Implement `Debug` for all public types
- Use `Cow<'_, str>` for string data that may be borrowed or owned

### Documentation
- All public items must have doc comments (`///`)
- Module-level documentation uses `//!`
- Document panics, safety requirements, and examples
- Use markdown in doc comments

Example:
```rust
//! PJRT Client
//!
//! This module provides the `Client` struct which represents a PJRT runtime instance.

/// Creates a buffer that carries an error future without allocating memory.
///
/// If this buffer is passed to an Execute call, the execution will fail
/// with the given error code and message.
pub fn create_error_buffer(...)
```

### Unsafe Code
- Minimize unsafe code blocks
- Always document safety requirements with `/// # Safety` section
- Mark unsafe functions with `unsafe fn`

### Builder Pattern
- Use `bon` crate for builder macros
- Use `#[builder(finish_fn = build)]` for custom finish function names
- Use `#[builder(start_fn)]` for required parameters in builder

Example:
```rust
#[bon]
impl Client {
    #[builder(finish_fn = build)]
    pub fn builder(
        #[builder(start_fn)] api: &Api,
        #[builder(default = bon::vec![], into)] options: Vec<NamedValue>,
    ) -> Result<Self> {
        // ...
    }
}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rai-explorers/pjrt-rs](https://github.com/rai-explorers/pjrt-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
