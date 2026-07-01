---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build, Test, and Development Commands

### Building
```bash
# Build in debug mode
cargo build

# Build optimized release binary
cargo build --release

# Check code without building (faster feedback)
cargo check
```

### Testing
```bash
# Run all tests
cargo test --all

# Run tests with output displayed
cargo test --all -- --nocapture

# Run a specific test
cargo test test_prompt -- --nocapture

# Run tests across all Rust versions (stable, beta, nightly)
cargo test --locked
```

### Code Quality
```bash
# Format code (required before commit)
cargo fmt --all

# Check formatting without making changes
cargo fmt --all -- --check

# Run clippy linter
cargo clippy --all-targets --all-features -- -D warnings

# Security audit dependencies
cargo audit
```

### Local Development
```bash
# Run the binary locally against current repo
./target/debug/aigitcommit

# Run with verbose output for debugging
./target/debug/aigitcommit -v

# Check environment configuration
cargo run -- --check-env

# Test API connectivity
cargo run -- --check-model
```

### Installation for Development
```bash
# Install locally for testing in other repos
cargo install --path .
```

## Code Architecture

### High-Level Overview

AIGitCommit is a CLI tool that generates semantic commit messages from staged Git changes using AI. The architecture follows a clear separation of concerns:

1. **CLI Layer** (`cli.rs`): Parses command-line arguments and options using clap
2. **Git Layer** (`git/`): Provides Git operations via git2 (diff, logs, commit)
3. **AI Layer** (`openai.rs`): Interfaces with OpenAI-compatible APIs
<<<<<<< HEAD
4. **Cache Layer** (`cache.rs`): Provides lightweight, stable FNV-1a-hashed on-disk caching of responses
5. **Utilities** (`utils.rs`): Shared utilities for output formatting, environment config, file operations

### Module Responsibilities

#### `main.rs` - Application Entry Point
- Orchestrates the workflow: parse args → get git info → call AI → format output → optionally commit
- Handles three main flows:
  - Hook installation (`install-hook` subcommand)
  - Environment/model checking (`--check-env`, `--check-model`)
  - Main workflow: analyze staged changes → generate message → display/commit
- Initializes tracing/logging based on verbosity flag (`-v`/`--verbose`)

#### `cli.rs` - Command-Line Interface
- Defines `Cli` struct with all command-line flags and options
- Supports subcommands: `install-hook` for git hook setup
- Options include: `--commit`, `--json`, `--copy-to-clipboard`, `--signoff`, `--yes`, `--no-cache`, `--clear-cache`, `--no-table`, `--save`, etc.
- Uses clap's derive macros for parsing and help text generation

#### `git/repository.rs` - Git Operations
- `Repository` struct wraps git2::Repository for high-level operations
- `get_diff()`: Gets staged changes as lines (`git diff --cached`)
- `get_logs(count)`: Gets recent commit messages for context on style/conventions
- `commit(&message)`: Creates a commit with the generated message
- `get_author()`: Retrieves author info from git config
- Filters out common noise (lock files, generated code) from diffs

#### `git/message.rs` - Commit Message Representation
- `GitMessage` struct represents a formatted commit message
- Follows Conventional Commits specification (type: scope: title + body)
- Supports Signed-off-by appending
- Handles serialization to different formats (JSON, plain text, table)

#### `openai.rs` - AI Integration
- `OpenAI` struct wraps async-openai's Client
- Configures API endpoint, authentication, proxy, and timeouts from environment
- `chat()`: Sends messages to OpenAI API and returns response
- `check_model()`: Verifies model availability before generating messages
- `prompt()`: Uses Askama templates to format user prompt with logs and diffs
- Template path: `templates/user.txt`

#### `cache.rs` - Response Cache Layer
- `Cache` struct manages file-based caching under `<repo>/.git/aigitcommit-cache/`
- Keys are derived from stable 64-bit FNV-1a hashing of variables that alter prompt inputs (model name, system prompt, staged diff lines, and recent git commit log messages).
- When staged diffs and surrounding context are unchanged, cached responses are reused immediately, avoiding redundant API calls
- Exposes options to bypass (`--no-cache`) or clear (`--clear-cache`)

#### `utils.rs` - Shared Utilities
- Environment variable reading with fallbacks
- Output formatting: `OutputFormat` enum supporting Table, JSON, and plain text
- Hook installation logic
- File I/O operations (save to file)
- Signoff configuration handling (env vars vs git config)

### Build System

- Uses Rust edition 2024
- `build.rs`: Generates `built.rs` (via `built` crate) containing build metadata (version, authors, etc.)
- Optimized release profile: LTO enabled, single codegen unit, stripped binary
- Key dependencies:
  - `git2`: Git operations
  - `async-openai`: OpenAI API client
  - `tokio`: Async runtime
  - `askama`: Template rendering for prompts
  - `clap`: CLI parsing
  - `tabled`: Table formatting
  - `cliclack`: Interactive prompts

### Environment Variables


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mingcheng/aigitcommit](https://github.com/mingcheng/aigitcommit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
