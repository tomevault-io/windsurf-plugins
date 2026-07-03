---
trigger: always_on
description: This file provides comprehensive guidance to agents when working with code in this repository.
---

# AGENTS.md

This file provides comprehensive guidance to agents when working with code in this repository.

## CI/CD Workflow & Code Gates

### Project Structure
- **Rust Edition**: 2021
- **Default Features**: server, stdio, macros, cache-memory, logging, api-key
- **Optional Features**: cache-redis, tls, auth, hyper-server, sse, streamable-http

### Build Commands

```bash
# Standard build
cargo build

# Build with all features
cargo build --all-features

# Build with specific feature
cargo build --features cache-redis

# Release build
cargo build --release
```

### Code Quality Gates (Must Pass Before Merge)

#### 1. Formatting Check
```bash
cargo fmt -- --check
```
- Checks if code is properly formatted
- Use `cargo fmt` to auto-fix formatting issues

#### 2. Clippy Linting
```bash
# Check without optional features
cargo clippy --all-targets -- -D warnings

# Check with all features (required - feature-gated code)
cargo clippy --all-features --all-targets -- -D warnings
```
**Critical**: Run both commands because Redis/auth code is feature-gated and checked separately.
- `-D warnings` treats all warnings as errors
- `--all-targets` includes lib, bins, tests, benches

#### 3. Security Audit
```bash
cargo install cargo-audit
cargo audit
```

### Testing Commands

#### Test Target Structure
Tests are split across THREE targets (not one suite):

```bash
# Unit tests (primary - new code goes here)
cargo test --test unit

# Legacy unit tests
cargo test --test unit_tests

# Integration tests
cargo test --test integration_tests

# E2E tests
cargo test --test e2e
```

#### Running Single Tests
**Important**: Use the correct target when running a single test:

```bash
# Correct: specify --test unit
cargo test --test unit test_oauth_config_github

# Correct: for tools_docs tests
cargo test --test unit test_lookup_crate_tool_execute_markdown

# Incorrect: will search all targets and may fail
cargo test test_oauth_config_github
```

#### Feature-Gated Testing

```bash
# Test with Redis cache (only when Redis is available)
cargo test --features cache-redis

# Test unit with Redis feature
cargo test --test unit --features cache-redis

# Test all features
cargo test --all-features
```

#### Multi-threaded Testing
Tests support concurrent execution. Default uses available cores:

```bash
# Use default threads
cargo test --test unit

# Force single thread (debugging)
cargo test --test unit -- --test-threads=1

# Use 4 threads
cargo test --test unit -- --test-threads=4
```

**Note**: Tests using `EnvVarGuard` or `serial_test` markers are handled for isolation.

### Documentation Gates

```bash
# Build documentation
cargo doc --no-deps --all-features

# Check documentation links
cargo doc --no-deps --all-features --document-private-items
```

### Coverage

```bash
cargo install cargo-tarpaulin
cargo tarpaulin --all-features --out Xml
```

## Code Style Guidelines

### Top-Level Directives (src/lib.rs)
```rust
#![warn(missing_docs)]      // Require documentation on public items
#![warn(clippy::pedantic)]    // Enable pedantic lints
#![allow(clippy::module_name_repetitions)]  // Allow mod name repetition
#![allow(clippy::missing_errors_doc)]  // Allow missing error docs
#![allow(clippy::missing_panics_doc)] // Allow missing panic docs
```

### Import Style

```rust
// External crates (sorted alphabetically)
use reqwest::Client;
use serde_json::json;
use tokio::spawn;

// Local modules (alphabetical, grouped by module)
use crate::cache::{Cache, CacheConfig};
use crate::error::{Error, Result};
use crate::tools::docs::DocService;
```

### Naming Conventions

- **Types/Structs**: PascalCase (e.g., `DocService`, `CacheConfig`)
- **Functions/Methods**: snake_case (e.g., `fetch_html`, `with_config`)
- **Constants**: SCREAMING_SNAKE_CASE (e.g., `VERSION`, `MAX_CONNECTIONS`)
- **Private fields**: snake_case (e.g., `client`, `cache`)
- **Modules**: snake_case (e.g., `cache`, `tools`, `utils`)

### Error Handling

**Library Code**: Use crate-specific Result/Error types:

```rust
// Define in src/error/mod.rs
#[derive(Error, Debug)]
pub enum Error {
    #[error("HTTP request failed: {0}")]
    Http(String),
}

// Use Result<T> alias
type Result<T> = std::result::Result<T, Error>;

// Convert foreign errors
map_err(|e| Error::http(e.to_string()))?;
```

**CLI Entry Points**: Return `Box<dyn std::error::Error>` for compatibility:

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    // CLI code
    Ok(())
}
```

### Public API Markers

```rust
// Mark functions that should use their return value
#[must_use]
pub fn create_config(&self) -> Config { }

// Mark test-only functions
#[cfg(test)]
pub fn test_helper() { }
```

### Async Patterns

```rust
// Async function with explicit lifetime
pub async fn fetch_data(&self, url: &str) -> Result<String> {
    let response = self.client.get(url).send().await?;
    response.text().await.map_err(Into::into)
}

// Use tokio::spawn for background tasks
tokio::spawn(async move {
    // Background work
});
```

### Config & Defaults

**Security**: Defaults are localhost-only:

```rust
// Default config restricts hosts/origins for security
impl Default for ServerConfig {
    fn default() -> Self {
        Self {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KingingWang/crates-docs](https://github.com/KingingWang/crates-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
