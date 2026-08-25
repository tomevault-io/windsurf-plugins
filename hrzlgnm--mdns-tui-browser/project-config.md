---
trigger: always_on
description: This file contains guidelines and commands for agentic coding agents working in this repository.
---

# AGENTS.md

This file contains guidelines and commands for agentic coding agents working in this repository.

## Build Commands

### Development
```bash
# Run the application in development mode
cargo run

# Run with custom service types
cargo run -- --service-types "_http._tcp.local.,_ssh._tcp.local."
```

### Building
```bash
# Build optimized release version
cargo build --release

# Build with audit trail (used in CI for releases)
cargo auditable build --release
```

### Testing
```bash
# Run all tests using nextest (preferred)
cargo nextest run --profile ci

# Run all tests using standard cargo test
cargo test

# Run a single test
cargo nextest run --profile ci test_name

# Run tests matching a pattern
cargo nextest run --profile ci test_pattern

# Run tests for a specific module
cargo nextest run --profile ci tui_app::tests
```

### Linting and Formatting
```bash
# Format code (will check CI)
cargo fmt

# Check formatting (fails in CI if not formatted)
cargo fmt -- --check

# Run clippy lints (fails in CI on warnings)
cargo clippy --tests -- -D warnings

# Check for typos
cargo install typos-cli
typos

# Check GitHub Actions workflows and reusable actions
actionlint

# Validate renovate configuration
docker run --rm --volume=$(pwd):$(pwd):ro --workdir=$(pwd) kokuwaio/renovate-config-validator:latest
```

## Code Style Guidelines

### Safety Policy
- **FORBIDDEN**: No `unsafe` blocks allowed anywhere in the codebase
- **REQUIRED**: `#![forbid(unsafe_code)]` at the top of every Rust file
- This is a **Safe Rust Only** project - memory safety is non-negotiable

### Imports and Dependencies
- Use `use` statements at the top of files in alphabetical order
- Group imports: std library, external crates, local modules
- Preferred libraries used in this project:
  - `ratatui` for TUI
  - `tokio` for async runtime
  - `crossterm` for terminal handling
  - `flume` for async channels
  - `mdns_sd` for service discovery
  - `clap` for CLI parsing
  - `chrono` for date/time handling

### Code Formatting
- Use `rustfmt` with default settings
- Maximum line length: 100 characters (rustfmt default)
- 4-space indentation (rustfmt default)
- Use `cargo fmt -- --check` to verify formatting

### Naming Conventions
- **Types**: `PascalCase` (structs, enums, type aliases)
- **Functions**: `snake_case`
- **Variables**: `snake_case`
- **Constants**: `SCREAMING_SNAKE_CASE`
- **Modules**: `snake_case`
- **Enums**: PascalCase for enum name, PascalCase for variants
- **Fields**: `snake_case` for struct fields

### Error Handling
- Use `Result<T, Box<dyn std::error::Error>>` for main functions
- Prefer `Option<T>` for values that may be absent
- Use `?` operator for error propagation
- Avoid panic! except in unrecoverable situations
- Use `unwrap()` only in tests or when absolutely certain

### Async/Concurrency Patterns
- Use `tokio::sync::RwLock` for shared state
- Use `flume` channels for async communication
- Mark async functions with `async`
- Use `.await` for async operations
- Prefer `Arc<RwLock<T>>` for shared mutable state

### Testing Guidelines
- Write unit tests in `#[cfg(test)]` modules
- Use descriptive test names following `test_functionality_expected_result` pattern
- Use `assert_eq!`, `assert!`, `assert_ne!` for assertions
- Test edge cases and error conditions
- Integration tests go in `tests/` directory (if present)
- Use `cargo nextest` for faster test execution
- Test configuration in `.config/nextest.toml`

### TUI Specific Patterns
- Use `ratatui` for all UI components
- Handle events with `crossterm::event`
- Use `ListState` for selection state management
- Separate UI logic from business logic
- Use `Frame<'_>` for rendering
- Follow the existing app structure: `AppState` for state, `run_tui` for main loop

### Documentation
- Add doc comments to public functions with `///`
- Use `///` for module-level documentation
- Include examples in doc comments when helpful
- Keep documentation concise and focused
- Document all key bindings in README

### Performance Considerations
- Use `--release` builds for performance testing
- Profile with appropriate tools if needed
- Consider allocation patterns in hot paths
- Use `BTreeMap`/`BTreeSet` when ordering matters
- Use `HashMap`/`HashSet` for O(1) lookups when order doesn't matter

## Project Structure

```
src/
├── main.rs          # Entry point, CLI argument parsing
├── tui_app.rs       # Main TUI application logic and tests
├── input.rs         # User input handling (filter, service type)
├── popup.rs         # Popup UI components (help, metrics)
├── scroll.rs        # Scroll state management
├── models.rs        # Data models
└── terminal.rs     # Terminal handling
```

## Development Workflow

1. REQUIRED: Create a branch for your changes with an appropriate prefix (e.g., `feat/`, `fix/`, `chore/`, `refactor/`, `docs/`)
2. Make changes to source code
3. Run `cargo fmt` to format code
4. Run `cargo clippy --tests -- -D warnings` to check for issues
5. Run `cargo nextest run --profile ci` to run tests
6. Run `cargo build --release` to build release version
7. Run `cargo clippy --release -- -D warnings` to ensure no warnings in release
 8. Run `actionlint` to check GitHub Actions workflows if modified

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hrzlgnm/mdns-tui-browser](https://github.com/hrzlgnm/mdns-tui-browser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
