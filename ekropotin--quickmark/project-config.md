---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

QuickMark is a lightning-fast Markdown/CommonMark linter written in Rust. It's inspired by markdownlint for Ruby and focuses on providing exceptional performance while integrating with development environments through LSP.

The original David Anson's markdownlint source code is available here: <https://github.com/DavidAnson/markdownlint>

## Project Structure

This is a Rust workspace with multiple crates implementing a clean separation of concerns:

```
quickmark/
├── Cargo.toml                 # Workspace configuration
├── crates/
│   ├── quickmark-core/        # Core linting logic with integrated configuration
│   │   ├── Cargo.toml
│   │   ├── src/
│   │   │   ├── lib.rs
│   │   │   ├── config/        # Configuration data structures and TOML parsing
│   │   │   ├── linter.rs      # Linting engine
│   │   │   ├── rules/         # Individual linting rules
│   │   │   ├── test_utils.rs  # Testing utilities
│   │   │   └── tree_sitter_walker.rs  # Tree-sitter AST traversal utilities
│   │   └── tests/
│   ├── quickmark-cli/         # CLI application
│   │   ├── Cargo.toml
│   │   └── src/
│   │       └── main.rs        # CLI interface
│   └── quickmark-server/      # Server application (LSP, etc.)
│       ├── Cargo.toml
│       └── src/
│           └── main.rs        # Server interface
├── docs/                      # Documentation
├── test-samples/              # Test files and configurations
└── vscode-quickmark/          # VSCode extension
```

## Common Commands

### Build and Development

- **Build all crates**: `cargo build`
- **Build release**: `cargo build --release`
- **Run tests**: `cargo test`
- **Run CLI linter**: `cargo run --bin qmark -- /path/to/file.md`
- **Run server**: `cargo run --bin quickmark-server`

### Configuration

- QuickMark looks for `quickmark.toml` in the current working directory
- If not found, default configuration is used

## Architecture

### Crate Responsibilities

**quickmark-core** (Core Library):

- Core linting logic with integrated configuration system
- TOML configuration parsing and validation
- Converts TOML structures to `QuickmarkConfig` objects  
- Tree-sitter based Markdown parsing
- Rule system with pluggable architecture
- Rule severity normalization and validation
- Self-contained design eliminates external configuration dependencies

**quickmark-cli** (CLI Application):

- Command-line interface using clap
- File I/O and user interaction
- Uses `quickmark-core` for configuration parsing and linting
- Parallel file processing with rayon
- File glob and ignore pattern support

**quickmark-server** (Server Application):

- LSP server interface for editor integration
- Uses `quickmark-core` for configuration and linting
- Async processing with tokio
- Real-time document analysis

### Core Components

**Linting Engine** (`quickmark-core/src/linter.rs`):

- `MultiRuleLinter`: Orchestrates multiple rule linters
- `RuleViolation`: Represents a linting error with location and message
- `Context`: Shared context containing file path and configuration
- Uses tree-sitter for Markdown parsing with tree-sitter-md grammar
- Filters rules based on severity configuration (off/warn/err)

**Configuration System** (`quickmark-core/src/config/mod.rs`):

- Format-agnostic configuration data structures
- `QuickmarkConfig`: Root configuration structure
- `RuleSeverity`: Enum for error/warning/off states
- `normalize_severities`: Validates and normalizes rule configurations
- No serialization dependencies - pure data structures

**TOML Configuration** (`quickmark-core/src/config/mod.rs`):

- Integrated TOML parsing within the core library
- `parse_toml_config`: Parses TOML strings into `QuickmarkConfig`
- `config_in_path_or_default`: Loads config from filesystem or defaults
- TOML-specific data structures with serde derives
- Direct conversion to core configuration types

**Rule System** (`quickmark-core/src/rules/mod.rs`):

- `Rule`: Static metadata structure defining rule properties
- `ALL_RULES`: Registry of all available rules
- Each rule implements `RuleLinter` trait with `feed` method
- Rules are dynamically instantiated based on configuration

### Linting Architecture Evolution

**Performance-Optimized Single-Pass Design**:

QuickMark has evolved from a simple node-based traversal to a sophisticated single-pass architecture that efficiently handles different rule types while maintaining exceptional performance. This design is inspired by the original markdownlint's architecture but leverages Rust's performance advantages and tree-sitter's robust parsing.

**Rule Type Classification**:

Rules are categorized into five types for optimal performance and implementation strategy:

- **Line-Based Rules** (e.g., MD013): Operate directly on raw text lines with AST context for configuration
- **Token-Based Rules** (e.g., MD001, MD003): Work with specific cached AST node types
- **Document-Wide Rules** (e.g., MD024, MD025): Require full document state analysis
- **Hybrid Rules** (e.g., MD022): Need both AST analysis and line context for structural spacing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ekropotin/quickmark](https://github.com/ekropotin/quickmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
