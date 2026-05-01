---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Agentic File Search is an AI-powered document search agent that explores files dynamically rather than using pre-computed embeddings. It uses a three-phase strategy: parallel scan, deep dive, and backtracking for cross-references. There is also an optional DuckDB-backed indexing pipeline for pre-indexed semantic+metadata retrieval.

**Tech Stack:** Python 3.10+, Google Gemini 3 Flash, LlamaIndex Workflows, Docling (document parsing), DuckDB (indexing), langextract (optional metadata extraction), FastAPI + WebSocket, Typer + Rich CLI.

## Common Commands

```bash
# Install dependencies
uv pip install .
uv pip install -e ".[dev]"  # with dev dependencies

# Run CLI (agentic exploration)
uv run explore --task "What is the purchase price?" --folder data/test_acquisition/

# Run CLI (indexed query - requires prior indexing)
uv run explore index data/test_acquisition/
uv run explore query --task "What is the purchase price?" --folder data/test_acquisition/

# Schema management
uv run explore schema discover data/test_acquisition/
uv run explore schema show data/test_acquisition/

# Run web UI
uv run uvicorn fs_explorer.server:app --host 127.0.0.1 --port 8000

# Run tests
uv run pytest                      # all tests
uv run pytest tests/test_fs.py     # single file
uv run pytest -k "test_name"       # single test

# Lint, format, typecheck (also available via Makefile)
uv run pre-commit run -a           # lint (or: make lint)
uv run ruff check .                # ruff only
uv run ruff format                 # format (or: make format)
uv run ty check src/fs_explorer/   # typecheck (or: make typecheck)
```

Entry points defined in `pyproject.toml`: `explore` → `fs_explorer.main:app`, `explore-ui` → `fs_explorer.server:run_server`.

## Architecture

### Core Flow (Agentic Mode)
```
User Query → Workflow (LlamaIndex) → Agent (Gemini) → Tools → Docling → Filesystem
```

### Core Flow (Indexed Mode)
```
User Query → Workflow → Agent → semantic_search/get_document → DuckDB → Ranked Results
```

### Key Modules (src/fs_explorer/)

- **workflow.py**: Event-driven orchestration using `llama-index-workflows`. Defines `FsExplorerWorkflow` with steps: `start_exploration`, `go_deeper_action`, `tool_call_action`, `receive_human_answer`. Uses singleton agent via `get_agent()`.

- **agent.py**: `FsExplorerAgent` manages Gemini API interaction. Chat history accumulates in `_chat_history`. `take_action()` sends history to LLM, receives structured JSON `Action`, auto-executes tool calls. `TokenUsage` tracks costs. Also contains the `TOOLS` registry (9 tools), `SYSTEM_PROMPT`, and indexed tool functions (`semantic_search`, `get_document`, `list_indexed_documents`). Index context is managed via module-level `set_index_context()`/`clear_index_context()`.

- **models.py**: Pydantic schemas for structured LLM output. `Action` contains one of: `ToolCallAction`, `GoDeeperAction`, `StopAction`, `AskHumanAction`. `Tools` TypeAlias defines all available tool names.

- **fs.py**: Filesystem operations. `scan_folder()` uses ThreadPoolExecutor for parallel document processing. `_DOCUMENT_CACHE` (dict) caches parsed documents keyed by `path:mtime`. Docling converts PDF/DOCX/PPTX/XLSX/HTML/MD to markdown.

- **main.py**: Typer CLI entry point with subcommands: default (agentic explore), `index`, `query`, `schema discover`, `schema show`.

- **server.py**: FastAPI server with WebSocket endpoint `/ws/explore` for real-time streaming.

- **exploration_trace.py**: Records tool call paths and extracts cited sources from final answers for the CLI summary.

### Indexing Subsystem (src/fs_explorer/indexing/)

- **pipeline.py**: `IndexingPipeline` orchestrates document parsing → chunking → metadata extraction → DuckDB upsert. Walks a folder for supported files, delegates to `SmartChunker` and `extract_metadata()`, handles schema resolution and deleted-file cleanup.

- **chunker.py**: `SmartChunker` splits parsed document text into overlapping chunks.

- **schema.py**: `SchemaDiscovery` auto-discovers metadata schemas from a corpus folder (file types, heuristic boolean fields like `mentions_currency`/`mentions_dates`). Optionally includes langextract fields.

- **metadata.py**: `extract_metadata()` produces per-document metadata dicts. Heuristic fields (filename, extension, document_type, currency/date detection) are always available. Optional langextract integration calls the `langextract` library for entity extraction (organizations, people, deal terms, etc.) via configurable profiles.

### Search Subsystem (src/fs_explorer/search/)

- **query.py**: `IndexedQueryEngine` runs parallel semantic (chunk text matching) + metadata (JSON filter) retrieval paths using ThreadPoolExecutor, then merges and ranks via `RankedDocument.combined_score`.

- **filters.py**: `parse_metadata_filters()` parses a human-readable filter DSL (`field=value`, `field>=num`, `field in (a, b)`, `field~substring`) into `MetadataFilter` objects. Validates against allowed schema fields.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PromtEngineer/agentic-file-search](https://github.com/PromtEngineer/agentic-file-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
