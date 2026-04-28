---
trigger: always_on
description: Ghost is a privacy-first AI desktop assistant built with Tauri v2 (Rust backend) + React/TypeScript/Vite (frontend). All AI inference runs locally via Candle + GGUF models. No cloud dependencies for core functionality.
---

# Ghost — Copilot Instructions

## Project Overview

Ghost is a privacy-first AI desktop assistant built with Tauri v2 (Rust backend) + React/TypeScript/Vite (frontend). All AI inference runs locally via Candle + GGUF models. No cloud dependencies for core functionality.

## Tech Stack

- **Framework:** Tauri v2
- **Backend:** Rust (edition 2021)
- **Frontend:** React + TypeScript + Vite
- **Database:** SQLite via rusqlite (FTS5 for full-text search)
- **ML:** Candle (Hugging Face) — all-MiniLM-L6-v2 for embeddings (384D), Qwen2.5-Instruct GGUF for generation
- **Agent:** ReAct engine with grammar-constrained tool calling
- **Protocols:** MCP (rmcp crate), AG-UI, A2UI, A2A
- **Licensing:** ghost-pro crate (freemium feature gating)

## Build & Test

```bash
# Build
cargo build --release

# Run in dev mode (Tauri)
cargo tauri dev

# Run tests
cargo test

# Run specific test
cargo test test_name

# Clippy lints
cargo clippy -- -D warnings

# Frontend dev
cd frontend && pnpm dev

# Frontend build
cd frontend && pnpm build
```

## Module Structure

```
src-tauri/
├── src/
│   ├── main.rs              # Tauri app entry, window setup
│   ├── lib.rs               # Core library exports
│   ├── agent/               # ReAct agent engine
│   │   ├── mod.rs            # Agent orchestrator
│   │   ├── react.rs          # ReAct loop (think → act → observe)
│   │   ├── grammar.rs        # Grammar-constrained generation
│   │   └── tools/            # Built-in tool implementations
│   ├── search/               # Hybrid search
│   │   ├── fts.rs            # FTS5 full-text search (<5ms)
│   │   ├── semantic.rs       # Vector search via Candle embeddings
│   │   └── hybrid.rs         # Combined ranking
│   ├── mcp/                  # MCP Server + Client
│   │   ├── server.rs         # Ghost as MCP server
│   │   ├── client.rs         # Connect to external MCP servers
│   │   └── catalog.rs        # 30+ curated + registry (6000+)
│   ├── extraction/           # File processing
│   │   ├── watcher.rs        # File system watcher
│   │   └── extractors/       # PDF, DOCX, 50+ formats
│   ├── models/               # Local AI model management
│   │   ├── loader.rs         # GGUF model loading via Candle
│   │   └── inference.rs      # Text generation pipeline
│   ├── db/                   # Database layer
│   │   ├── mod.rs            # Connection pool, migrations
│   │   ├── schema.rs         # Table definitions
│   │   └── migrations/       # SQL migrations
│   └── pro/                  # ghost-pro crate integration
│       ├── license.rs        # License key validation
│       └── features.rs       # Feature gating logic
├── Cargo.toml
└── tauri.conf.json
```

## Rust Conventions

- **Error handling:** Use `anyhow::Result` for application errors, `thiserror` for library errors. Never `unwrap()` in production paths.
- **Async:** Tokio runtime (Tauri v2 default). Use `#[tauri::command]` for frontend-callable functions.
- **Tauri commands:** All Tauri commands go in dedicated command modules, registered in `main.rs` via `invoke_handler`.
- **Database:** All SQL goes through rusqlite with parameterized queries. Never interpolate strings into SQL.
- **Logging:** Use `tracing` crate with structured fields. `tracing::info!`, `tracing::error!`, etc.
- **Testing:** Unit tests in the same file (`#[cfg(test)]` module). Integration tests in `tests/` directory.

## How to Add a New Tool

Tools are functions the ReAct agent can call during reasoning.

1. Create the tool implementation in `src/agent/tools/`:
```rust
// src/agent/tools/my_tool.rs
use serde::{Deserialize, Serialize};
use anyhow::Result;

#[derive(Debug, Deserialize)]
pub struct MyToolInput {
    pub query: String,
}

#[derive(Debug, Serialize)]
pub struct MyToolOutput {
    pub result: String,
}

pub async fn execute(input: MyToolInput) -> Result<MyToolOutput> {
    // Implementation here
    Ok(MyToolOutput { result: "done".into() })
}
```

2. Register in `src/agent/tools/mod.rs`:
```rust
pub mod my_tool;

// Add to the tool registry
registry.register("my_tool", ToolDef {
    name: "my_tool",
    description: "What this tool does",
    parameters: serde_json::json!({
        "type": "object",
        "properties": {
            "query": { "type": "string", "description": "The search query" }
        },
        "required": ["query"]
    }),
    handler: |input| Box::pin(my_tool::execute(serde_json::from_value(input)?)),
});
```

3. Add tests:
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[tokio::test]
    async fn test_my_tool_basic() {
        let input = MyToolInput { query: "test".into() };
        let output = execute(input).await.unwrap();
        assert!(!output.result.is_empty());
    }
}
```

4. Run `cargo test` and `cargo clippy` before committing.

## How to Add a New MCP Server

MCP servers extend Ghost's capabilities via the Model Context Protocol.

1. Add to curated catalog in `src/mcp/catalog.rs`:
```rust
CatalogEntry {
    id: "my-server",
    name: "My Server",
    description: "What it does",
    transport: Transport::Stdio {
        command: "npx",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghostapp-ai/ghost](https://github.com/ghostapp-ai/ghost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
