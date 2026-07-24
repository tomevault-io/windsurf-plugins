---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pommel is a local-first semantic code search system designed to reduce context window consumption for AI coding agents. It maintains an always-current vector database of code embeddings, enabling targeted semantic searches instead of reading numerous files into context.

**Status:** v0.7.3 - Configurable timeouts for cold starts and slow connections

## Code Search Priority

**IMPORTANT: Use `pm search` BEFORE using Grep/Glob for code exploration.**

When looking for:
- How something is implemented → `pm search "authentication flow"`
- Where a pattern is used → `pm search "error handling"`
- Related code/concepts → `pm search "database connection"`
- Code that does X → `pm search "validate user input"`

Only fall back to Grep/Glob when:
- Searching for an exact string literal (e.g., a specific error message)
- Looking for a specific identifier name you already know
- Pommel daemon is not running

Example workflow:
```bash
# First: semantic search to find relevant code
pm search "retry logic with backoff" --limit 5

# Then: read the specific files/lines returned
# Only use grep if you need exact string matching
```

## Quick Start

```bash
# Initialize Pommel in a project
pm init --auto --claude --start

# Search the codebase semantically
pm search "database connection handling"
pm search "error handling patterns" --limit 5
pm search "CLI command setup" --json

# Check status
pm status

# Reindex after major changes
pm reindex
```

## Architecture

```
AI Agent / Developer
        │
        ▼
    Pommel CLI (pm)  ──────► search, status, init, start, stop, reindex, config
        │
        ▼
    Pommel Daemon (pommeld)
    ├── File watcher (fsnotify, debounced)
    ├── Tree-sitter chunker (AST-aware)
    └── Embedding generator (Ollama client)
        │
        ▼
    SQLite + sqlite-vec (local vector DB)
        ▲
        │
    Jina Code Embeddings via Ollama (768-dim vectors)
```

**Data flows:**
1. **Indexing:** File changes → Watcher → Chunker → Embedder → Vector DB
2. **Search:** Query → CLI → Daemon → Embedder → Vector DB → Ranked results

## CLI Commands

| Command | Description |
|---------|-------------|
| `pm init` | Initialize Pommel in project (flags: `--auto`, `--claude`, `--start`) |
| `pm search <query>` | Semantic search (flags: `--json`, `--limit`, `--level`, `--path`) |
| `pm status` | Show daemon status and index statistics |
| `pm start` | Start the daemon |
| `pm stop` | Stop the daemon |
| `pm reindex` | Force full reindex of the codebase |
| `pm config` | View/modify configuration |
| `pm version` | Show version information |

All commands support `--json` for structured output and `-p/--project` to specify project root.

## Technology Stack

| Component | Technology |
|-----------|------------|
| Language | Go 1.21+ |
| Platform Support | macOS (Intel/ARM), Linux (x64/ARM64), Windows (x64/ARM64) |
| CLI Framework | Cobra + Viper |
| Vector Database | SQLite + sqlite-vec |
| Embedding Model | jina-embeddings-v2-base-code (via Ollama) |
| Code Parsing | Tree-sitter (33 languages - see Supported Languages below) |
| File Watching | fsnotify |
| HTTP Server | go-chi |

## Project Structure

```
pommel/
├── cmd/
│   ├── pm/              # CLI entry point
│   └── pommeld/         # Daemon entry point
├── internal/
│   ├── api/             # HTTP API types and handlers
│   ├── chunker/         # Tree-sitter based code chunking
│   ├── cli/             # Cobra command implementations
│   ├── config/          # YAML configuration loading/validation
│   ├── daemon/          # Daemon server, watcher, indexer
│   ├── db/              # SQLite + sqlite-vec database layer
│   ├── embedder/        # Ollama embedding client
│   ├── models/          # Shared data models
│   ├── search/          # Vector similarity search
│   └── setup/           # Dependency detection
├── scripts/
│   └── install.sh       # Installation script
├── docs/                # Documentation and plans
└── .pommel/             # Per-project data (gitignored)
    ├── config.yaml      # Project configuration
    └── pommel.db        # SQLite database with vectors
```

## Development

### Prerequisites

- Go 1.24+
- Ollama with `unclemusclez/jina-embeddings-v2-base-code` model

### Building

**Important:** Always include `-tags fts5` to enable FTS5 full-text search support.

```bash
# Using make (recommended - includes all required tags)
make build

# Or manually with required tags
go build -tags fts5 -o pm ./cmd/pm
go build -tags fts5 -o pommeld ./cmd/pommeld
```

### Testing

**Important:** Always include `-tags fts5` to enable FTS5 support in tests.

```bash
# Using make (recommended)
make test

# Or manually with required tags
go test -tags fts5 ./...                    # Run all tests
go test -tags fts5 ./internal/cli/...       # Run CLI tests
go test -tags fts5 -v -run TestInitCmd      # Run specific test
```

### Installation

**macOS / Linux:**
```bash
curl -fsSL https://raw.githubusercontent.com/dbinky/Pommel/main/scripts/install.sh | bash
```

**Windows (PowerShell):**
```powershell

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dbinky/Pommel](https://github.com/dbinky/Pommel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
