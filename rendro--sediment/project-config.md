---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Build
cargo build --release

# Run MCP server (default)
cargo run --release

# Run with verbose logging
cargo run --release -- --verbose

# Run CLI commands
cargo run --release -- init                     # Initialize project
cargo run --release -- stats                    # Show database statistics
cargo run --release -- list                     # List stored items
cargo run --release -- list --scope global      # List global items
cargo run --release -- store "some content"     # Store content
cargo run --release -- store -                  # Store from stdin
cargo run --release -- recall "search query"    # Semantic search
cargo run --release -- forget <id>              # Delete an item
cargo run --release -- --json list              # JSON output (any command)

# Run tests (requires model download on first run)
cargo test
cargo test -- --ignored              # Include tests that require model download

# Install locally
cargo install --path .
```

## Architecture

Sediment is a semantic memory system for AI agents, running as an MCP (Model Context Protocol) server. It combines vector search, a property graph, and access tracking into a unified memory intelligence layer.

### Two-Database Hybrid (all local, embedded, zero config)

- **LanceDB** — Vector embeddings + semantic similarity (items and chunks)
- **SQLite** (`access.db`) — Graph relationships, access tracking, decay scoring, consolidation queue

### Core Components

- **`src/main.rs`** - CLI entry point with subcommands (init, stats, list, store, recall, forget) and MCP server startup
- **`src/lib.rs`** - Library root exposing public API, project detection, scope types, and project ID migration
- **`src/db.rs`** - LanceDB wrapper handling vector storage, hybrid search (vector + FTS/BM25), and CRUD operations
- **`src/embedder.rs`** - Local embeddings using `all-MiniLM-L6-v2` via Candle (384-dim vectors)
- **`src/chunker.rs`** - Smart content chunking by type (markdown, code, JSON, YAML, text)
- **`src/document.rs`** - ContentType enum for routing content to the appropriate chunker
- **`src/item.rs`** - Unified Item, Chunk, SearchResult, StoreResult, and ConflictInfo types
- **`src/access.rs`** - SQLite-based access tracking, validation counting, and memory decay scoring
- **`src/graph.rs`** - SQLite graph store: relationship tracking (RELATED, SUPERSEDES, CO_ACCESSED, CLUSTER_SIBLING edges)
- **`src/consolidation.rs`** - Background consolidation: auto-merging near-duplicates, linking similar items
- **`src/error.rs`** - SedimentError enum with typed error variants (Database, Embedding, Arrow, etc.)
- **`src/retry.rs`** - Retry utilities with exponential backoff (3 attempts, 100ms–2s)

### MCP Server (`src/mcp/`)

- **`mod.rs`** - Module exports
- **`server.rs`** - stdio JSON-RPC server with shared embedder, graph path, consolidation semaphore
- **`tools.rs`** - 4 MCP tools: `store`, `recall`, `list`, `forget`
- **`protocol.rs`** - MCP protocol types and JSON-RPC handling

### Data Flow

1. **Store**: Content → Embedder (384-dim vector) → LanceDB storage → Graph node creation → Conflict detection → Consolidation queue
2. **Chunking**: Long content (>1000 chars) → Type-aware splitting → Individual chunk embeddings
3. **Recall**: Query → Embedder → Vector similarity search → Project boosting → Decay scoring → Trust-weighted re-ranking → Graph backfill → 1-hop graph expansion → Co-access suggestions → Cross-project flagging → Background consolidation + co-access recording
4. **Consolidation** (background): Queue candidates → >=0.95 similarity: merge (delete old, transfer edges, SUPERSEDES edge) → 0.85-0.95: link (RELATED edge)
5. **Clustering** (periodic): Triangle detection in graph → CLUSTER_SIBLING edges

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `SEDIMENT_DB` | `~/.sediment/data` | Override database path. Also available as `--db` CLI flag. Useful for ephemeral/isolated storage: `SEDIMENT_DB=/tmp/task sediment store "..."` |
| `SEDIMENT_EMBEDDING_MODEL` | `all-MiniLM-L6-v2` | Override embedding model. Options: `all-MiniLM-L6-v2`, `bge-small-en-v1.5` |

Priority: `--db` flag > `SEDIMENT_DB` env var > default `~/.sediment/data`.

### Key Design Decisions

- **Two-database hybrid**: LanceDB for vectors, SQLite for graph relationships + mutable counters
- **Single central database** at `~/.sediment/data/` stores all projects; graph + access at `~/.sediment/access.db`
- **Project scoping** via UUID stored in `.sediment/config` per project
- **Similarity boosting**: Same-project items unchanged, different projects get 0.875x penalty (12.5% spread)
- **Hybrid search**: Vector similarity combined with FTS/BM25 scoring. BM25 boost is additive (max 0.12, power-law gamma 2.0). FTS index rebuilt on each store
- **Conflict detection**: Items with >=0.85 similarity flagged on store and enqueued for consolidation
- **Fresh DB connection per tool call** with shared embedder for efficiency

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rendro/sediment](https://github.com/rendro/sediment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
