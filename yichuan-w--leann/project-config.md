---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LEANN is a lightweight vector database and RAG (Retrieval-Augmented Generation) system that achieves 97% storage reduction compared to traditional vector databases through graph-based selective recomputation. It enables semantic search across various data sources (emails, browser history, chat history, code, documents) on a single laptop without cloud dependencies.

## Build & Development Commands

### Quick install (pip)

```bash
pip install leann
```

### Development setup (from source)

```bash
# Install uv first (required package manager)
curl -LsSf https://astral.sh/uv/install.sh | sh

git submodule update --init --recursive

# macOS
brew install libomp boost protobuf zeromq pkgconf
uv sync

# Ubuntu/Debian
sudo apt-get install libomp-dev libboost-all-dev protobuf-compiler \
    libabsl-dev libmkl-full-dev libaio-dev libzmq3-dev
uv sync

# Windows (requires VS 2022 Build Tools with C++ workload, vcpkg, chocolatey)
choco install cmake swig pkgconfiglite nuget.commandline -y
vcpkg install zeromq:x64-windows openblas:x64-windows lapack:x64-windows boost-program-options:x64-windows protobuf:x64-windows
# Set CMAKE_PREFIX_PATH, PKG_CONFIG_PATH, OPENBLAS_LIB to vcpkg paths (see README)
uv sync --extra diskann

# Install lint tools
uv sync --group lint

# Install test tools
uv sync --group test
```

## Code Quality

```bash
# Format code
ruff format

# Lint with auto-fix
ruff check --fix

# Pre-commit hooks (install once)
pre-commit install

# Run pre-commit manually
uv run pre-commit run --all-files
```

## Architecture

### Core API Layer (`packages/leann-core/src/leann/`)

- `api.py`: Main APIs - `LeannBuilder`, `LeannSearcher`, `LeannChat`
- `react_agent.py`: `ReActAgent` for multi-turn reasoning
- `cli.py`: CLI implementation (`leann build`, `leann search`, `leann ask`)
- `chat.py`: LLM provider integrations (OpenAI, Ollama, HuggingFace, Anthropic)
- `embedding_compute.py`: Embedding computation (sentence-transformers, MLX, OpenAI)
- `metadata_filter.py`: Search result filtering by metadata

### Backend Layer (`packages/`)

- `leann-backend-hnsw/`: Default backend using FAISS HNSW for fast in-memory search
- `leann-backend-ivf/`: IVF backend (FAISS IndexIVFFlat + DirectMap.Hashtable) supporting in-place add/remove without rebuild
- `leann-backend-diskann/`: DiskANN backend for larger-than-memory datasets
- `leann-mcp/`: MCP server for Claude Code integration

Backends are auto-discovered via `leann-backend-*` naming convention and registered in `registry.py`.

### RAG Applications (`apps/`)

Example applications demonstrating RAG on various data sources:
- `document_rag.py`: PDF/TXT/MD documents
- `email_rag.py`: Apple Mail
- `browser_rag.py`: Chrome browser history
- `wechat_rag.py`, `imessage_rag.py`: Chat history
- `code_rag.py`: Codebase search with AST-aware chunking
- `slack_rag.py`, `twitter_rag.py`: MCP-based live data

## Key Design Patterns

### Incremental Update (IVF backend)

The IVF backend supports in-place updates and deletes without rebuilding the entire index:
- `add_vectors(index_path, embeddings, passage_ids)`: Append new vectors to an existing index.
- `remove_ids(index_path, passage_ids)`: Remove vectors by passage ID using FAISS DirectMap.Hashtable.
- `LeannBuilder.update_index()`: High-level API that orchestrates remove-then-add for changed files, compacts `passages.jsonl`, and updates the offset map.

`leann build` is idempotent — re-running it on an existing index automatically performs an incremental update instead of a full rebuild. It detects new, modified, and removed files and applies the minimal set of changes:
- **IVF**: Supports add, remove, and modify incrementally (remove old chunks then re-insert).
- **HNSW** (non-compact): Supports add-only incremental updates; modified/removed files trigger a full rebuild.
- Use `--force` / `-f` to force a full rebuild regardless.

### Index Structure

A LEANN index consists of:
- `<name>.meta.json`: Metadata (backend, embedding model, dimensions)
- `<name>.passages.jsonl`: Raw text chunks with metadata
- `<name>.passages.idx`: Offset map for fast passage lookup
- `<name>.index`: Backend-specific vector index

### Embedding Recomputation

The core storage optimization: instead of storing embeddings, LEANN stores a pruned graph and recomputes embeddings on-demand during search via ZMQ server communication.

## CLI Usage

```bash
# Build index
leann build my-docs --docs ./documents/

# Search
leann search my-docs "query"

# Interactive chat
leann ask my-docs --interactive

# List indexes
leann list

# Remove index
leann remove my-docs
```

## Common Development Tasks

Running example RAG applications:
```bash
# Document RAG (easiest to test)
python -m apps.document_rag --query "What is LEANN?"

# Code RAG
python -m apps.code_rag --repo-dir ./src --query "How does search work?"
```

## Python Version

Requires Python 3.10+ (uses PEP 604 union syntax `X | Y`).




# Agent Coding Guidelines

## General
- Voice input may contain typos — interpret intent, not literal text.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yichuan-w/LEANN](https://github.com/yichuan-w/LEANN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
