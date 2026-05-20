---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

rag-client is a flexible Python tool for Retrieval-Augmented Generation (RAG) that augments LLM interactions with contextual document retrieval. It supports both ephemeral (file-cached) and persistent (Postgres-based) modes, and is designed for integration with local/open-weight models.

## Package Structure

The project is organized as a Python package with a modular architecture:

```
rag_client/                      # Main package directory
├── core/                        # Core RAG functionality
│   ├── workflow.py             # RAGWorkflow class for orchestration
│   ├── models.py               # QueryState and ChatState management
│   ├── retrieval.py            # CustomRetriever implementations
│   └── indexing.py             # Document indexing logic
├── config/                      # Configuration system
│   └── models.py               # Dataclass-based YAML configs
├── cli/                         # Command-line interface
│   ├── __init__.py            # CLI setup and parsing
│   └── commands.py            # Command handlers (index, search, query, chat, serve)
├── api/                         # REST API implementation
│   └── server.py              # FastAPI OpenAI-compatible server
├── providers/                   # LLM and embedding providers
│   └── factory.py             # Provider factory pattern
├── storage/                     # Storage backends
│   ├── ephemeral.py           # File-based caching
│   └── postgres.py            # PostgreSQL with pgvector
├── utils/                       # Helper utilities
│   ├── logging.py             # Centralized logging
│   ├── readers.py             # Custom document readers (OrgReader, MailParser)
│   └── helpers.py             # General utilities
├── exceptions.py               # Custom exception hierarchy
└── types.py                    # Type definitions and aliases

Legacy/Compatibility:
├── main.py                     # CLI entry point
├── rag.py                      # Legacy monolithic implementation (deprecated)
├── rag_compat.py              # Compatibility layer
├── chat.py                     # Custom SimpleContextChatEngine
└── api.py                      # Legacy API server (deprecated)

Configuration Examples:
├── examples/configs/
│   ├── basic.yaml             # Simple Ollama setup
│   ├── openai.yaml            # OpenAI configuration
│   └── postgres.yaml          # Persistent storage setup
├── chat.yaml                  # Main chat configuration
└── guidance.yaml              # Alternative example config

Documentation & Tests:
├── docs/                       # Sphinx documentation
├── tests/                      # Test suite
└── query-test.sh              # Integration test script
```

## Core Components

### Main Entry Point
- **main.py**: CLI argument parsing and command dispatch
  - Uses `typed-argparse` for type-safe argument handling
  - Supports: index, search, query, chat, serve commands
  - Configurable logging levels (--verbose, --debug)

### RAG Workflow (`rag_client/core/workflow.py`)
- **RAGWorkflow** class orchestrates the entire RAG pipeline:
  - `load_config()`: YAML configuration loading with validation
  - `load_retriever()`: Vector index creation and loading
  - `retrieve_nodes()`: Semantic search execution
  - Supports ephemeral (file-cached) and persistent (Postgres) modes
  - Automatic fingerprinting for cache management

### State Management (`rag_client/core/models.py`)
- **QueryState**: Manages query engine and LLM for one-shot Q&A
- **ChatState**: Manages chat engine, memory, and conversation history
- State persistence to `~/.config/rag-client/chat_store.json`

### API Server (`rag_client/api/server.py`)
- OpenAI-compatible FastAPI server with endpoints:
  - `POST /v1/chat/completions`: Chat completions (streaming supported)
  - `POST /v1/completions`: Text completions
  - `POST /v1/embeddings`: Generate embeddings
  - `GET /v1/models`: List available models
- Features: API key auth, CORS middleware, request/response logging

### Custom Components

#### SimpleContextChatEngine (`chat.py`)
- Performance-optimized alternative to standard ContextChatEngine
- Uses COMPACT response mode instead of iterative refinement
- Better throughput for large top_k values
- Limits context assembly to configured window size

#### Custom Readers (`rag_client/utils/readers.py`)
- **OrgReader**: Parses Emacs org-mode files
  - Splits by org entries (headings)
  - Preserves heading and property metadata
  - Body-only text extraction
- **MailParser**: Parses .eml email files
  - Extracts From, Date, Subject headers
  - Plain text body extraction
  - Metadata preservation

## CLI Commands

### Command Reference

```bash
# General format
python main.py --config <yaml-file> [options] <command> [args]
```

#### Global Options
- `--config, -c`: Path to YAML configuration file (required)
- `--from`: Input source (file, directory, or `-` for stdin)
- `--recursive`: Recursively process directories
- `--num-workers, -j`: Parallel workers for document processing
- `--verbose`: Show progress and INFO logs
- `--debug`: Show DEBUG logs and detailed traces

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jwiegley/rag-client](https://github.com/jwiegley/rag-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
