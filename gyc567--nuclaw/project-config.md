---
trigger: always_on
description: This file provides guidance for AI coding agents working on NuClaw.
---

# AGENTS.md

This file provides guidance for AI coding agents working on NuClaw.

## Project Overview

NuClaw is a personal Claude assistant that:
- Connects to WhatsApp/Telegram for messaging
- Runs AI agents in isolated containers (Docker/Apple Container)
- Supports scheduled tasks with cron expressions
- Uses SQLite for persistent storage (rusqlite + r2d2)
- Provides built-in skills and Provider/Channel registry system

## Build Commands

```bash
# Debug build (default)
cargo build

# Production build
cargo build --release

# Type checking only
cargo check

# Linting with clippy
cargo clippy

# Run clippy with auto-fix
cargo clippy --fix

# Generate documentation
cargo doc --open

# Format code
cargo fmt

# Check formatting
cargo fmt --check
```

## Testing Commands

```bash
# Run all tests
cargo test

# Run a single test module
cargo test skills::
cargo test providers::
cargo test channels::

# Run a single test by name
cargo test test_parse_container_output

# Run tests with output
cargo test -- --nocapture

# Run integration tests only
cargo test --test integration_tests

# Run tests and show coverage (requires tarpaulin)
cargo tarpaulin --no-fail-in --out Html
```

## Module Structure

```
src/
├── main.rs              # CLI entry point
├── lib.rs               # Library exports and module declarations
├── agent_runner.rs      # Agent execution (container + API modes)
├── channels.rs          # Channel trait and registry
├── config.rs            # Configuration and env loading
├── container_runner.rs  # Docker/Apple Container spawning
├── db.rs               # SQLite database operations
├── error.rs            # NuClawError enum
├── logging.rs          # Logging setup
├── providers.rs        # Provider registry (LLM providers)
├── skills.rs           # Built-in skills system
├── task_scheduler.rs   # Cron-based task scheduling
├── telegram.rs         # Telegram Bot API integration
├── types.rs            # Core data structures
├── utils.rs            # JSON/file utilities
├── whatsapp.rs         # WhatsApp MCP integration
├── orchestrator/       # Task queue and execution orchestration
├── telegram/           # Telegram submodules (client, types, utils)
├── context/            # Context management (cache, loader, security)
├── workflow/           # Workflow configuration and hooks
└── skill_creator/      # Skill creation (intent, eval, writer)
```

## Code Style Guidelines

### Imports

```rust
// Standard ordering: std → external → crate
use std::collections::HashMap;
use std::sync::RwLock;

use async_trait::async_trait;
use serde::{Deserialize, Serialize};

use crate::error::{NuClawError, Result};
```

### Error Handling

```rust
// Use thiserror for error enums
#[derive(Error, Debug)]
pub enum NuClawError {
    #[error("Database error: {message}")]
    Database { message: String },
    
    #[error("API error: {message}")]
    Api { message: String },
}

// Use ? operator and map_err
async fn fetch_data() -> Result<String> {
    let response = client.get(url).send().await
        .map_err(|e| NuClawError::Api {
            message: format!("Request failed: {}", e)
        })?;
    Ok(response)
}

// Implement From traits for ergonomic error conversion
impl From<std::io::Error> for NuClawError {
    fn from(e: std::io::Error) -> Self {
        NuClawError::FileSystem { message: e.to_string() }
    }
}
```

### Async Functions

```rust
// All async functions return Result
async fn process_message(&self) -> Result<Option<String>> {
    // Implementation
}

// Use async_trait for trait methods
#[async_trait]
pub trait AgentRunner: Send + Sync {
    async fn run(&self, input: Input) -> Result<Output>;
}
```

### Naming Conventions

- **Modules**: `snake_case` (e.g., `agent_runner`, `task_scheduler`)
- **Types**: `PascalCase` (e.g., `ScheduledTask`, `ContainerInput`)
- **Functions/Variables**: `snake_case` (e.g., `run_container`, `max_output_size`)
- **Constants**: `SCREAMING_SNAKE_CASE` (e.g., `DEFAULT_TIMEOUT_MS`)
- **Type Parameters**: `PascalCase` (e.g., `T`, `Result<T, E>`)

### Types and Structs

```rust
// Simple structs with derive
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ContainerInput {
    pub prompt: String,
    pub session_id: Option<String>,
    pub group_folder: String,
}

// Use Option for optional fields
// Use Result for fallible operations
// Use HashMap for dynamic key-value collections
```

### Registry Pattern (Two-Step Registration)

```rust
// Step 1: Define spec in PROVIDERS constant
pub const PROVIDERS: &[ProviderSpec] = &[
    ProviderSpec::new("anthropic", "ANTHROPIC_API_KEY", ...),
];

// Step 2: Use environment variables for configuration
// No code changes needed - auto-configured!
```

### Trait Design

```rust
// Keep traits small and focused
#[async_trait]
pub trait Channel: Send + Sync {
    fn name(&self) -> &str;
    async fn send(&self, jid: &str, message: &str) -> Result<()>;
    async fn start(&self) -> Result<()>;
}

// Use Send + Sync for thread-safe shared state
```

### Testing Requirements

```rust
// All new modules must have 100% test coverage
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_function_behavior() {
        // Test normal path
    }

    #[test]
    fn test_edge_cases() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gyc567/nuclaw](https://github.com/gyc567/nuclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
