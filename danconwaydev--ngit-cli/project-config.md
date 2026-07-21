---
trigger: always_on
description: Documentation for AI agents and automated tools working with the ngit codebase.
---

# AGENTS.md

Documentation for AI agents and automated tools working with the ngit codebase.

## Project Overview

**ngit** is a nostr plugin for git that enables decentralized code collaboration using the nostr protocol.

- **Language**: Rust
- **Type**: Command-line tool with git integration
- **Architecture**: Two main binaries (`ngit` and `git-remote-nostr`) with shared library code
- **Key Dependencies**: nostr-sdk, git2, clap, tokio

### Core Concepts

1. **Nostr Integration**: Uses nostr for repository identification, discovery, state management, and collaboration (PRs, issues)
2. **Git Server Agnostic**: Still requires git servers for data storage, but they're interchangeable
3. **Decentralized**: Multiple relays and git servers can be used for redundancy
4. **URL Format**: `nostr://<npub|nip05-address>/<identifier>`

## Project Structure

```
ngit/
├── src/
│   ├── bin/
│   │   ├── git_remote_nostr/    # Git remote helper implementation
│   │   │   ├── capabilities.rs
│   │   │   ├── fetch.rs
│   │   │   ├── list.rs
│   │   │   ├── main.rs
│   │   │   └── push.rs
│   │   └── ngit/                # Main CLI tool
│   │       ├── main.rs
│   │       └── sub_commands/
│   └── lib/                     # Shared library code
│       ├── git/                 # Git operations
│       ├── login/               # User authentication
│       ├── client.rs            # Nostr client
│       ├── fetch.rs             # Fetch operations
│       ├── git_events.rs        # Git event handling
│       ├── list.rs              # List operations
│       ├── push.rs              # Push operations
│       ├── repo_ref.rs          # Repository references
│       ├── repo_state.rs        # Repository state
│       └── utils.rs             # Utilities
├── tests/                       # Integration tests
├── test_harness/                # e2e test utility
└── git_hooks/                   # Git hooks for development
```

## Key Files and Their Purposes

### Core Library (`src/lib/`)

- **`repo_ref.rs`**: Repository reference handling, URL parsing, grasp server detection
  - Functions: `is_grasp_server_in_list()`, `is_grasp_server_clone_url()`, `normalize_grasp_server_url()`
  - Critical for identifying and validating repository URLs
- **`repo_state.rs`**: Manages repository state (refs, commits, etc.)

- **`client.rs`**: Nostr client wrapper and relay communication

- **`git_events.rs`**: Handles git-related nostr events

- **`push.rs`**: Push operations to git servers and nostr relays

- **`fetch.rs`**: Fetch operations from git servers

### Binaries

- **`git-remote-nostr`**: Git remote helper that enables git to work with nostr:// URLs
- **`ngit`**: Main CLI tool for managing nostr repositories

## Development Guidelines

### Code Style

- Follow Rust standard formatting (`rustfmt.toml` is configured)
- Run `cargo fmt` before committing
- Run `cargo clippy` to catch common issues
- Use `anyhow::Result` for error handling
- Prefer explicit error messages with context

### Testing

```bash
# Run all tests
cargo test

# Run specific test
cargo test test_name

# Run tests with output
cargo test -- --nocapture

# Run integration tests only
cargo test --test ngit_init
```

### Important Testing Notes

1. **URL Comparison**: When comparing git server URLs, be aware of normalization:
   - `is_grasp_server_in_list()`: Compares full URLs (including repo path)
   - `normalize_grasp_server_url()`: Extracts server part only (strips npub and path)

2. **Test Structure**: Integration tests are in `tests/`, unit tests are in module files

### Building

```bash
# Development build
cargo build

# Release build
cargo build --release

# Binaries will be in target/debug/ or target/release/
# - ngit
# - git-remote-nostr
```

## Common Tasks for Agents

### Adding New Features

1. **Identify the correct module**:
   - Git operations → `src/lib/git/`
   - Nostr operations → `src/lib/client.rs`, `src/lib/git_events.rs`
   - CLI commands → `src/bin/ngit/sub_commands/`
   - Remote helper → `src/bin/git_remote_nostr/`

2. **Add tests**: Always add tests for new functionality

3. **Update error handling**: Use `anyhow::Context` for error messages

4. **Check dependencies**: Avoid adding unnecessary dependencies

### Debugging Issues

1. **Check test failures**: Run `cargo test` to identify failing tests

2. **Review error context**: Look at the full error chain with `.context()`

3. **Examine URL handling**: Many issues relate to URL parsing/normalization
   - Check `repo_ref.rs` for URL-related functions
   - Verify grasp server detection logic

4. **Trace nostr events**: Check event creation and publishing in `git_events.rs`

### Refactoring

1. **Maintain backward compatibility**: This is a CLI tool users depend on

2. **Update all call sites**: Use `cargo check` to find all references

3. **Run full test suite**: Ensure nothing breaks

4. **Check integration tests**: These test real-world scenarios

## Architecture Patterns

### Error Handling

```rust
use anyhow::{Context, Result};

fn example() -> Result<()> {
    something()
        .context("descriptive error message")?;
    Ok(())
}
```

### Async Operations

- Uses `tokio` runtime for async operations
- Nostr operations are async
- Git operations are mostly sync


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DanConwayDev/ngit-cli](https://github.com/DanConwayDev/ngit-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
