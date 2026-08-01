---
trigger: always_on
description: Quick reference for AI coding agents working on cfn-teleport.
---

# AGENTS.md

Quick reference for AI coding agents working on cfn-teleport.

## Project Overview

**cfn-teleport** is a Rust CLI tool that moves CloudFormation resources between stacks using AWS SDK for Rust. It features interactive prompts (dialoguer), async AWS operations (tokio), and integrates with CloudFormation-based integration tests.

## Build/Test/Lint Commands

### Building

- `make build` - Build debug binary
- `make release` - Build optimized release binary
- `cargo check` - Fast compilation check without producing binary
- `cargo install --path .` - Install binary locally from source

### Testing

- `cargo test --all` - Run all unit tests (if any exist)
- `cargo test <test_name>` - Run a specific test by name (e.g., `cargo test split_ids`)
- `cargo test <module>::<test>` - Run specific test in module
- `make test` - Full test suite:
  - Runs `cargo check`
  - Runs `cargo test --all`
  - Deploys CloudFormation test stacks (requires AWS credentials)
  - Runs integration tests with actual AWS resources

### Linting & Formatting

- `cargo fmt` - Auto-format code
- `cargo fmt -- --check` - Check formatting without modifying files
- `cargo clippy` - Run linter with default warnings
- `cargo clippy -- -D warnings` - Run linter treating all warnings as errors (CI mode)
- `make lint` - Run both fmt check and clippy with `-D warnings` (CI equivalent)

### Running

- `cargo run` - Run the program in debug mode
- `cargo run -- --help` - Run with arguments (e.g., help)
- `cargo run -- --source Stack1 --target Stack2` - Run with specific options
- `make run` - Same as `cargo run`

### CloudFormation Integration Tests

- `cd test/cloudformation && make deploy` - Deploy all test stacks to AWS
- `cd test/cloudformation && make verify-formats` - Verify stack template formats (JSON vs YAML)
- `cd test/cloudformation && make DESTROY` - Destroy all test stacks
- `make test-clean-all` - Clean up all test resources by tags (S3, DynamoDB, EC2, etc.)
- `make test-reset` - Full reset: destroy stacks + clean all tagged resources

## Code Style & Conventions

### Rust Edition & Tooling

- **Edition**: Rust 2021
- **Formatting**: Use `cargo fmt` (enforced by CI via `cargo fmt -- --check`)
- **Lints**: Clippy with `-D warnings` - **all warnings are treated as errors** in CI
- **Minimum Supported Rust Version (MSRV)**: Test against stable and nightly toolchains

### Import Organization

Group imports in this order (see `main.rs:1-11`):

1. External crate imports (alphabetical)
2. Local module declarations (`mod`)
3. Local module imports (`use crate::`)

Example:

```rust
use aws_config::BehaviorVersion;
use aws_sdk_cloudformation as cloudformation;
use clap::Parser;
use dialoguer::{Confirm, Input, MultiSelect};
use std::collections::HashMap;
use std::error::Error;
use std::io;
mod spinner;
mod supported_resource_types;
```

### Naming Conventions

- **Functions & variables**: `snake_case` (e.g., `get_stacks`, `source_stack`)
- **Types & structs**: `PascalCase` (e.g., `Args`, `Spin`)
- **Constants**: `SCREAMING_SNAKE_CASE` (e.g., `DEMO`, `SUPPORTED_RESOURCE_TYPES`)
- **Lifetimes**: Single lowercase letter (e.g., `'a`)

### Error Handling

- **Main & fallible functions**: Return `Result<T, Box<dyn Error>>` for flexibility
- **AWS SDK operations**: Return `Result<T, cloudformation::Error>` for specific AWS errors
- **Error propagation**: Use `?` operator for concise error handling
- **Custom errors**: Convert to `Box<dyn Error>` with `.into()` or string conversion
- **User-facing errors**: Use `eprintln!` for error messages with helpful context (see `main.rs:63-73`)
- **Process exit**: Use `process::exit(1)` for fatal errors after printing user-friendly messages

Example:

```rust
async fn get_stacks(
    client: &cloudformation::Client,
) -> Result<Vec<cloudformation::types::StackSummary>, cloudformation::Error> {
    let resp = client.list_stacks().send().await?;
    Ok(resp.stack_summaries.unwrap_or_default())
}
```

### Type Annotations

- **Function signatures**: Always use explicit types for parameters and return values
- **Function bodies**: Use type inference (`let var = ...`) for local variables when clear
- **Complex types**: Consider type aliases for readability

### Async/Await Patterns

- **Entry point**: Use `#[tokio::main]` on `async fn main()`
- **AWS operations**: All AWS SDK calls are `async` - use `.await?` pattern
- **Dependencies**: `tokio = { version = "1.37.0", features = ["full"] }`
- **Threading**: Use `std::thread::sleep` for blocking waits (see `main.rs:895`)

### Documentation

- Use `///` for public API documentation comments
- Use `//` for inline explanatory comments
- Use `//!` for module-level documentation
- CLI arg documentation: Use `#[arg(...)]` doc attributes (see `main.rs:19-33`)

### Code Organization

- **Modules**: Separate concerns into modules (`spinner.rs`, `supported_resource_types.rs`)
- **Function size**: Keep functions focused; extract helpers (e.g., `split_ids`, `user_confirm`)
- **Constants**: Extract magic values and large static data to module-level constants

### User Interaction Patterns

- **Prompts**: Use `dialoguer` crate for interactive prompts (`Select`, `MultiSelect`, `Confirm`, `Input`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [udondan/cfn-teleport](https://github.com/udondan/cfn-teleport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
