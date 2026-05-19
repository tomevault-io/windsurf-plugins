---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Documentation

- **[docs/architecture.md](docs/architecture.md)** - Complete system architecture, module structure, data flow, and design patterns.
- **[docs/adding-executors.md](docs/adding-executors.md)** - Step-by-step guide for adding new AI CLI tool support.

## Extended Guidelines

For comprehensive best practices, reference these files in `.cursor/rules/`:

- **`ratatui.mdc`** - Complete Ratatui 0.30.0 field guide covering widgets, layouts, styling, testing, and immediate-mode architecture. Read this when working on UI components, layouts, or widget implementations.

- **`rust.mdc`** - Comprehensive Rust 2024 edition best practices covering code organization, error handling, performance, security, and testing. Read this when making architectural decisions, handling errors, or optimizing code.

## Project Overview

**McGravity** is a TUI-based interface for orchestrating AI-assisted coding tools (Claude Code, OpenAI Codex CLI, Gemini CLI, etc.). It provides a unified workflow called "McGravity Flow" that helps software engineers:

- Compose better-structured tasks for AI coding assistants
- Execute tasks across multiple AI CLI tools with consistent patterns
- Track completed tasks and provide context for planning next steps
- Manage file system operations with proper sandboxing and approval flows
- Review and approve AI-generated changes before application

The project is inspired by OpenAI's Codex CLI architecture, which uses a modular Rust workspace design with separated concerns for core logic, TUI presentation, and file system operations.

## Build Commands

```bash
cargo build          # Build the project
cargo run            # Run the application
cargo test           # Run tests
cargo test <name>    # Run a specific test
cargo clippy         # Run linter
cargo fmt            # Format code
```

## Local Installation

To install mcgravity locally to `~/.cargo/bin/` (making it available system-wide):

```bash
cargo install --path . --force
```

- The `--force` (`-f`) flag ensures the binary is rebuilt and replaced even if already installed
- After installation, run from anywhere with: `mcgravity`
- To uninstall: `cargo uninstall mcgravity`

**When the user asks to "install mcgravity locally" or "install this package", run:**

```bash
cargo install --path . --force
```

## Mandatory Quality Checks

**IMPORTANT: After every code change, run the full validation pipeline:**

```bash
cargo fmt && cargo clippy && cargo build && cargo test
```

This ensures:

1. **Formatting** (`cargo fmt`) - Consistent code style
2. **Linting** (`cargo clippy`) - Catches common mistakes and enforces best practices
3. **Build** (`cargo build`) - Verifies compilation succeeds
4. **Tests** (`cargo test`) - Ensures no regressions

### CI Pipeline Compliance

**CRITICAL: All changes must pass the CI pipeline defined in [`.github/workflows/ci.yaml`](.github/workflows/ci.yaml).**

Before considering any change complete, verify it will pass CI by running the exact checks from the workflow:

```bash
# Format check (must pass with no changes needed)
cargo fmt --all -- --check

# Clippy with strict warnings-as-errors
cargo clippy --all-targets --all-features -- -D warnings

# Build and test with all features and locked dependencies
cargo build --locked --all-features
cargo test --locked --all-features
```

The CI runs on every PR and push to main. A change is not complete until it passes all CI checks.

### Clippy Guidelines

Run clippy with strict settings for maximum code quality:

```bash
# Standard check
cargo clippy

# Strict mode - treat warnings as errors
cargo clippy -- -D warnings

# With additional lints for pedantic checks
cargo clippy -- -W clippy::pedantic -W clippy::nursery
```

#### Clippy Best Practices

- **Fix all warnings**: Never ignore clippy warnings; they often indicate real issues
- **Use `#[allow(...)]` sparingly**: Only suppress warnings with a comment explaining why
- **Common lints to watch for**:
  - `clippy::unwrap_used` - Prefer `expect()` with context or proper error handling
  - `clippy::clone_on_ref_ptr` - Avoid unnecessary clones
  - `clippy::large_enum_variant` - Box large enum variants
  - `clippy::needless_pass_by_value` - Use references when ownership isn't needed
  - `clippy::missing_errors_doc` - Document error conditions in public APIs
  - `clippy::missing_panics_doc` - Document panic conditions

## Project Structure

```
mcgravity/
├── CLAUDE.md                    # This file - AI assistant guidance
├── Cargo.toml                   # Project manifest
├── Cargo.lock                   # Dependency lock file
│
├── docs/                        # Project documentation
│   ├── architecture.md          # System architecture and design patterns
│   └── adding-executors.md      # Guide for adding new AI CLI tools
│
├── .cursor/rules/               # IDE-specific guidelines
│   ├── ratatui.mdc              # Ratatui 0.30.0 best practices
│   └── rust.mdc                 # Rust 2024 edition best practices
│
├── src/
│   ├── main.rs                  # Entry point, terminal setup, event loop

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tigranbs/mcgravity](https://github.com/tigranbs/mcgravity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
