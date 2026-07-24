---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

rlm_rs is a Rust crate built with modern tooling, strict type safety, and zero-cost abstractions.

## Project Structure

```
src/
├── lib.rs           # Library entry point and public API
├── main.rs          # Binary entry point (optional)
├── error.rs         # Error types (if separated)
└── ...              # Additional modules

tests/
└── integration_test.rs  # Integration tests

benches/              # Benchmarks (with criterion)
examples/             # Example programs
```

## Build Commands

This project uses [Cargo](https://doc.rust-lang.org/cargo/) as the build system.

```bash
# Build the project
cargo build

# Build with optimizations
cargo build --release

# Run tests
cargo test

# Run tests with output
cargo test -- --nocapture

# Run specific test
cargo test test_name

# Run benchmarks
cargo bench

# Run linting
cargo clippy --all-targets --all-features

# Format code
cargo fmt

# Check formatting
cargo fmt -- --check

# Generate documentation
cargo doc --open

# Check supply chain security
cargo deny check

# Run with MIRI (undefined behavior detection)
cargo +nightly miri test

# Run all checks (lint + format + test + doc + deny)
cargo fmt -- --check && cargo clippy --all-targets --all-features -- -D warnings && cargo test && cargo doc --no-deps && cargo deny check
```

## Code Style Requirements

This project uses **clippy** with pedantic and nursery lints, and **rustfmt** for formatting.

### Key Rules

- **Line length**: 100 characters
- **Edition**: 2024
- **MSRV**: 1.95
- **Unsafe code**: Forbidden unless explicitly justified
- **Panics**: Not allowed in library code (`unwrap`, `expect`, `panic!`)

### Error Handling

Always use `Result` types for fallible operations. Never panic in library code:

```rust
// Good - Returns Result
pub fn parse(input: &str) -> Result<Value, ParseError> {
    if input.is_empty() {
        return Err(ParseError::EmptyInput);
    }
    // parsing logic
    Ok(value)
}

// Bad - Panics
pub fn parse(input: &str) -> Value {
    input.parse().unwrap() // Never do this in library code
}
```

Use `thiserror` for custom error types:

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum Error {
    #[error("invalid input: {0}")]
    InvalidInput(String),

    #[error("operation failed")]
    OperationFailed {
        #[source]
        source: std::io::Error,
    },
}
```

### Documentation

All public items must have documentation with examples:

```rust
/// Processes the input data according to the configuration.
///
/// # Arguments
///
/// * `input` - The data to process.
/// * `config` - Processing configuration.
///
/// # Returns
///
/// The processed result.
///
/// # Errors
///
/// Returns [`Error::InvalidInput`] if the input is malformed.
///
/// # Examples
///
/// ```rust
/// use rlm_rs::{process, Config};
///
/// let result = process("data", &Config::default())?;
/// assert!(!result.is_empty());
/// # Ok::<(), rlm_rs::Error>(())
/// ```
pub fn process(input: &str, config: &Config) -> Result<Output, Error> {
    // implementation
}
```

### Ownership and Borrowing

Prefer borrowing over ownership:

```rust
// Good - borrows
pub fn process(data: &[u8]) -> Vec<u8> { ... }

// Avoid - takes ownership unnecessarily
pub fn process(data: Vec<u8>) -> Vec<u8> { ... }
```

Use `Cow` for flexible string handling:

```rust
use std::borrow::Cow;

pub fn normalize(s: &str) -> Cow<'_, str> {
    if s.contains(' ') {
        Cow::Owned(s.replace(' ', "_"))
    } else {
        Cow::Borrowed(s)
    }
}
```

### Builder Pattern

Use builder pattern for complex configuration:

```rust
#[derive(Debug, Clone, Default)]
pub struct Config {
    timeout: Duration,
    retries: u32,
}

impl Config {
    #[must_use]
    pub const fn new() -> Self {
        Self {
            timeout: Duration::from_secs(30),
            retries: 3,
        }
    }

    #[must_use]
    pub const fn with_timeout(mut self, timeout: Duration) -> Self {
        self.timeout = timeout;
        self
    }

    #[must_use]
    pub const fn with_retries(mut self, retries: u32) -> Self {
        self.retries = retries;
        self
    }
}
```

## Testing Conventions

- **Unit tests**: Inside `src/*.rs` with `#[cfg(test)]` modules
- **Integration tests**: `tests/` directory
- **Doc tests**: Examples in documentation
- **Property tests**: Use `proptest` for property-based testing

### Test Structure

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_success_case() {
        let result = function_under_test(valid_input);
        assert_eq!(result, expected_output);
    }

    #[test]
    fn test_error_case() {
        let result = function_under_test(invalid_input);
        assert!(matches!(result, Err(Error::InvalidInput(_))));
    }
}
```

### Property-Based Testing

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn property_holds(input in any::<i64>()) {
        prop_assert!(predicate(input));
    }
}
```

## Linting Configuration

Clippy is configured to deny:
- `unwrap_used`, `expect_used`, `panic` - Use Result instead

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zircote/rlm-rs](https://github.com/zircote/rlm-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
