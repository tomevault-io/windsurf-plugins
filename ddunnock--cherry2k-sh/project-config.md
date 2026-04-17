---
trigger: always_on
description: > **Project**: Cherry2K.sh - Zsh Terminal AI Assistant
---

# CLAUDE.md

> **Project**: Cherry2K.sh - Zsh Terminal AI Assistant
> **Language**: Rust (≥1.93, Edition 2024)
> **Workflow**: Get Shit Done (GSD) Claude Code

---

## Overview

Cherry2K.sh is a zsh terminal-based AI assistant built in Rust with a provider-agnostic architecture. It supports multiple AI backends (OpenAI, Anthropic, Ollama) through a unified trait abstraction, uses SQLite for conversation persistence, and integrates into zsh via pure shell functions and ZLE widgets.

## Quick Start

```bash
# Build and install
cargo build --release
./install.sh  # Sets up zsh integration

# Development
cargo check                    # Fast type checking
cargo clippy -- -D warnings    # Lint with warnings as errors
cargo test                     # Run tests
cargo llvm-cov --fail-under-lines 80  # Coverage check
```

## Project Structure

```
cherry2k/
├── crates/
│   ├── core/                   # Domain logic + provider abstraction
│   │   ├── src/
│   │   │   ├── lib.rs
│   │   │   ├── provider/       # AI provider trait + implementations
│   │   │   │   ├── mod.rs
│   │   │   │   ├── trait.rs    # AiProvider trait definition
│   │   │   │   ├── openai.rs
│   │   │   │   ├── anthropic.rs
│   │   │   │   └── ollama.rs
│   │   │   ├── conversation/   # Conversation management
│   │   │   ├── config/         # Configuration handling
│   │   │   └── error.rs        # Error types (thiserror)
│   │   └── Cargo.toml
│   ├── storage/                # SQLite persistence (rusqlite)
│   │   ├── src/
│   │   │   ├── lib.rs
│   │   │   ├── schema.rs       # Database schema
│   │   │   ├── migrations/     # SQL migrations
│   │   │   └── repository.rs   # Data access layer
│   │   └── Cargo.toml
│   └── cli/                    # Terminal interface
│       ├── src/
│       │   ├── main.rs
│       │   ├── commands/       # CLI commands
│       │   ├── repl/           # Interactive mode
│       │   └── output/         # Terminal formatting
│       └── Cargo.toml
├── zsh/                        # Zsh integration
│   ├── cherry2k.plugin.zsh     # Main plugin file
│   ├── widgets/                # ZLE widget functions
│   └── completions/            # Zsh completions
├── .claude/
│   └── standards/              # Project standards (see below)
├── Cargo.toml                  # Workspace root
└── CLAUDE.md                   # This file
```

## Standards Reference

**CRITICAL**: Always consult these standards before implementation:

| File                                                   | When to Read                              |
|--------------------------------------------------------|-------------------------------------------|
| [constitution.md](.claude/standards/constitution.md)   | Always - global principles, quality gates |
| [rust.md](.claude/standards/rust.md)                   | All Rust development                      |
| [testing.md](.claude/standards/testing.md)             | Writing tests                             |
| [security.md](.claude/standards/security.md)           | API keys, secrets, input validation       |
| [git-cicd.md](.claude/standards/git-cicd.md)           | Commits, branches, PRs                    |
| [documentation.md](.claude/standards/documentation.md) | Doc comments, README updates              |

## GSD Workflow

This project uses **Get Shit Done (GSD)** methodology with Claude Code:

1. **Before Starting**: Run `/gsd:progress` to check current state
2. **Planning**: Use `/gsd:plan-phase` for feature planning
3. **Execution**: Use `/gsd:execute-phase` for implementation
4. **Debugging**: Use `/gsd:debug` for systematic issue resolution
5. **Verification**: Always verify with `/gsd:verify-work`

### GSD Commands Reference

```bash
/gsd:new-project      # Initialize project roadmap
/gsd:plan-phase       # Plan implementation phase
/gsd:execute-phase    # Execute planned work
/gsd:debug           # Systematic debugging
/gsd:verify-work     # Verify completion
/gsd:progress        # Check overall progress
```

## Architecture

### AI Provider Abstraction

The core architecture centers on a provider-agnostic trait:

```rust
/// Core AI provider trait - all providers MUST implement this.
#[async_trait]
pub trait AiProvider: Send + Sync {
    /// Send a message and receive a streaming response.
    async fn complete(&self, request: CompletionRequest) -> Result<CompletionStream, ProviderError>;

    /// Provider identifier for logging/config.
    fn provider_id(&self) -> &'static str;

    /// Validate configuration before use.
    fn validate_config(&self) -> Result<(), ConfigError>;
}
```

### Provider Implementations

| Provider  | API Type     | Auth    | Features                          |
|-----------|--------------|---------|-----------------------------------|
| OpenAI    | REST         | API Key | GPT-4, GPT-3.5, streaming         |
| Anthropic | REST         | API Key | Claude models, streaming          |
| Ollama    | REST (local) | None    | Local models, no network required |

### SQLite Storage

Uses `rusqlite` with migrations for:
- Conversation history
- User preferences
- Provider configurations
- Session metadata

**Homebrew Integration**: SQLite installed via `brew install sqlite3` for optimal macOS performance.

### Zsh Integration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ddunnock) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
