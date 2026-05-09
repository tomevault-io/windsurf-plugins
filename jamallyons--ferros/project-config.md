---
trigger: always_on
description: Ferros is a Rust-native debugger built as a workspace of multiple crates. Each crate has a specific responsibility:
---


# Ferros Rust Development Rules

## Project Overview
Ferros is a Rust-native debugger built as a workspace of multiple crates. Each crate has a specific responsibility:
- `ferros`: Command-line interface
- `ferros-core`: Low-level debugging primitives and process control
- `ferros-mir`: MIR integration, introspection, and analysis
- `ferros-ui`: Optional TUI/GUI for visualization
- `ferros-protocol`: Communication layer between debugger and frontend
- `ferros-utils`: Shared utilities, logging, config, and helpers

## Code Style & Formatting

### Formatting Rules
- Always run `cargo fmt --all` before committing
- Follow the project's `rustfmt.toml`

### Linting Rules
- Always run `cargo clippy --all -- -D warnings` before committing
- Workspace-level lint rules:
  - `unsafe_code = "forbid"` - NO unsafe code allowed unless absolutely necessary and thoroughly documented
  - `unused = "warn"` - Clean up unused code
- Prefer explicit types where clarity helps
- Use `clippy::pedantic` suggestions when appropriate

## Workspace Configuration

### Crate Structure
- All crates live under `/crates` directory
- Each crate should have its own `README.md` describing its purpose and public API
- All crates inherit workspace lints via `[lints] workspace = true`

### Dependencies
- **Async Runtime**: Prefer `tokio` for async operations
- **Error Handling**: Use `thiserror` for library errors, `anyhow` for application errors
- **Logging**: Use `tracing` and `tracing-subscriber` for structured logging
- **Serialization**: Use `serde` and `serde_json` for data serialization
- **CLI**: Use `clap` for command-line argument parsing
- **Testing**: Use `proptest` for property-based testing, `insta` for snapshot testing

## Error Handling

### Error Types
- Use `thiserror` for library crates (`ferros-core`, `ferros-mir`, `ferros-protocol`)
- Use `anyhow` for application crates (`ferros`, `ferros-ui`)
- Always provide context with `.context()` or `#[source]` attributes
- Use `Result<T, E>` for fallible operations
- Prefer `?` operator over explicit `match` when propagating errors

### Error Messages
- Write clear, actionable error messages
- Include relevant context (file paths, line numbers, variable values when safe)
- Use structured error types with `#[error("...")]` attributes

## Async Programming

### Async Patterns
- Use `tokio` as the async runtime
- Prefer `async fn` over manual `Future` implementations
- Use `tokio::spawn` for concurrent tasks
- Use `tokio::select!` for concurrent operations
- Prefer `tokio::sync::mpsc` for channels
- Use `Arc` and `Mutex`/`RwLock` for shared state in async contexts

### Async Safety
- Ensure all async operations are properly awaited
- Use `#[tokio::test]` for async tests
- Be mindful of cancellation and cleanup in async code

## Documentation

### Public API Documentation
- Document all public functions, types, and modules with Rustdoc comments (`///`)
- Use `# Examples` sections for complex APIs
- Include `# Safety` sections for any unsafe code (should be rare)
- Use `# Errors` sections to document error conditions
- Use `# Panics` sections if functions can panic

### Code Comments
- Write comments that explain "why" not "what"
- Use `//` for inline comments
- Use `/* */` for multi-line comments when needed
- Keep comments up-to-date with code changes

## Testing

### Test Organization
- Each crate contains its own `tests/` directory for unit tests
- Top-level `tests/` directory for integration tests across crates
- Use `#[cfg(test)]` modules for unit tests within source files

### Test Practices
- Write tests for all new functionality
- Use descriptive test names: `test_function_name_scenario_expected_result`
- Use `#[should_panic]` sparingly and document why
- Use `criterion` for performance benchmarking
- Run `cargo test --workspace --all-features` before committing

### Platform-Scoped Tests and Examples
- **CRITICAL**: All platform-specific tests and examples MUST be scoped with `#[cfg(target_os = "...")]`
- This ensures CI works on all platforms (Linux, macOS, Windows) without failing
- **For Examples**:
  - Wrap platform-specific code in `#[cfg(target_os = "...")]` modules
  - Provide a stub `main()` for non-target platforms that prints a helpful error message
  - Example pattern:
    ```rust
    #[cfg(not(target_os = "macos"))]
    fn main() {
        eprintln!("This example is macOS-only.");
        std::process::exit(1);
    }
    
    #[cfg(target_os = "macos")]
    mod macos_impl {
        // ... platform-specific code ...
    }
    
    #[cfg(target_os = "macos")]
    fn main() {
        macos_impl::main();
    }
    ```
- **For Tests**:
  - Use `#[cfg(target_os = "...")]` on test functions or modules
  - Ensure tests compile on all platforms (even if they're skipped)
  - Example: `#[cfg(target_os = "macos")] #[test] fn test_macos_feature() { ... }`
- **Why this matters**: CI runs on Linux, so macOS/Windows-specific code must compile but can be skipped

### Test Data
- Use `proptest` for property-based testing
- Use `insta` for snapshot testing when appropriate
- Create test fixtures in `tests/fixtures/` when needed

## Crate-Specific Guidelines

### ferros-core
- Focus on low-level debugging primitives

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JamalLyons/ferros](https://github.com/JamalLyons/ferros) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
