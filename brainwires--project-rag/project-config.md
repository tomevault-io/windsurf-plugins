---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Project RAG is a Rust-based Model Context Protocol (MCP) server that provides AI assistants with RAG (Retrieval-Augmented Generation) capabilities for understanding massive codebases. It uses FastEmbed for local embeddings and Qdrant for vector storage, enabling semantic search across indexed codebases.

**Key Technology Stack:**
- Rust 2024 edition with async/await (Tokio)
- MCP protocol via `rmcp` crate (v0.8) with macros
- FastEmbed (all-MiniLM-L6-v2 model, 384 dimensions)
- LanceDB vector database (default, embedded) or Qdrant (optional, external server)
- Tantivy BM25 keyword search with Reciprocal Rank Fusion (RRF) for hybrid search
- Tree-sitter AST-based chunking for 12 languages
- Persistent hash cache for incremental updates across restarts
- File walking with .gitignore support via `ignore` crate

## Essential Commands

### Building and Running
```bash
# Build debug version
cargo build

# Build optimized release version
cargo build --release

# Quick compile check without building
cargo check

# Run the MCP server over stdio
cargo run
# Or directly:
./target/release/project-rag
```

### Testing
```bash
# Run all unit tests (386 tests across all modules)
cargo test --lib

# Run tests for specific module
cargo test --lib types::tests
cargo test --lib chunker::tests
cargo test --lib cache::tests
cargo test --lib bm25_search::tests

# Run with verbose output
cargo test --lib -- --nocapture

# Run tests with debug logging
RUST_LOG=debug cargo test --lib -- --nocapture
```

### Code Quality
```bash
# Format code (required before commits)
cargo fmt

# Check lints with clippy
cargo clippy

# Auto-fix clippy suggestions
cargo clippy --fix
```

### Debugging
```bash
# Run with debug logging
RUST_LOG=debug cargo run

# Run with trace-level logging
RUST_LOG=trace cargo run
```

### Vector Database Management

**Default (LanceDB - Embedded)**
```bash
# No external setup required - LanceDB is embedded
# Data stored in ./.lancedb directory by default
```

**Optional (Qdrant - External Server)**
```bash
# Build with Qdrant backend
cargo build --release --no-default-features --features qdrant-backend

# Start Qdrant via Docker
docker run -p 6333:6333 -p 6334:6334 \
    -v $(pwd)/qdrant_data:/qdrant/storage \
    qdrant/qdrant

# Check Qdrant health
curl http://localhost:6334/health

# View Qdrant logs
docker logs <container-id>
```

## Architecture

### Core Design Principles

1. **Modular Trait-Based Design**: Each major component is defined by a trait (EmbeddingProvider, VectorDatabase) with concrete implementations, enabling easy swapping of backends.

2. **MCP Protocol Integration**: Uses `rmcp` macros (`#[tool]`, `#[prompt]`, `#[tool_router]`, `#[prompt_router]`) to define 9 MCP tools and 9 slash commands. The server communicates over stdio following MCP spec.

3. **Async-First Architecture**: Built on Tokio runtime with async traits. File walking runs on blocking threads via `tokio::task::spawn_blocking` to avoid blocking the async runtime.

4. **Smart Indexing with Auto-Detection**: The `index_codebase` tool automatically detects whether to perform full indexing (new codebase) or incremental updates (previously indexed). Tracks file hashes (SHA256) in persistent cache (`.cache/project-rag/hash_cache.json`) to detect changes across server restarts.

5. **Hybrid Search**: Combines vector similarity (semantic understanding) with BM25 keyword matching using Reciprocal Rank Fusion (RRF) for optimal search results. Tantivy provides full-text search capabilities with IDF scoring.

### Module Structure

```
src/
├── mcp_server.rs           # Main MCP server with 9 tools + 9 prompts
│   ├── RagMcpServer        # Server state (embedding provider, vector DB, chunker, hash cache)
│   ├── Tool handlers       # index_codebase (smart), query_codebase, find_definition, etc.
│   └── Prompt handlers     # Slash commands for each tool
├── client/                 # High-level client API
│   ├── mod.rs              # RagClient - unified interface for all operations
│   └── indexing/           # Indexing pipeline with progress reporting
├── embedding/              # Embedding generation abstraction
│   ├── mod.rs              # EmbeddingProvider trait
│   └── fastembed_manager.rs # FastEmbed implementation (unsafe workaround for mutability)
├── vector_db/              # Vector database abstraction
│   ├── mod.rs              # VectorDatabase trait
│   ├── lance_client.rs     # LanceDB implementation (default, embedded)
│   └── qdrant_client.rs    # Qdrant implementation (optional, external server)
├── indexer/                # File processing and chunking
│   ├── mod.rs              # Module exports
│   ├── file_walker.rs      # Directory traversal with .gitignore support
│   ├── chunker.rs          # Code chunking (FixedLines, SlidingWindow, AST-based)
│   └── ast_parser.rs       # Tree-sitter AST parsing for 12 languages
├── relations/              # Code relationship analysis (definitions, references, call graphs)
│   ├── mod.rs              # RelationsProvider trait, HybridRelationsProvider

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Brainwires/project-rag](https://github.com/Brainwires/project-rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
