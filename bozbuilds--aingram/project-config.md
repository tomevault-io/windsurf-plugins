---
trigger: always_on
description: AIngram is a local-first, privacy-first agent memory system built on SQLite and sqlite-vec. All state lives in a single `.db` file — no cloud services, no external APIs at runtime. Core primitives are: typed `MemoryEntry` records, a knowledge graph (entities + relationships), reasoning chains, ONNX vector embeddings (Nomic MiniLM, 768-dim), and QJL 1-bit auxiliary vectors for coarse search (`vec_entries_qjl`). AIngram is **not** a general-purpose vector database, not a cloud memory service, and 
---

# AIngram — Developer Reference for AI Coding Agents

## Project Identity

AIngram is a local-first, privacy-first agent memory system built on SQLite and sqlite-vec. All state lives in a single `.db` file — no cloud services, no external APIs at runtime. Core primitives are: typed `MemoryEntry` records, a knowledge graph (entities + relationships), reasoning chains, ONNX vector embeddings (Nomic MiniLM, 768-dim), and QJL 1-bit auxiliary vectors for coarse search (`vec_entries_qjl`). AIngram is **not** a general-purpose vector database, not a cloud memory service, and not an ORM.

## Dev Environment

- Python ≥ 3.11 required
- Install all dependencies: `pip install -e ".[dev]"`
- The ONNX embedding model (Nomic MiniLM) is downloaded to `~/.aingram/models/` on first instantiation of `MemoryStore` if not already cached. Tests that require embeddings inject a mock embedder to avoid this download — don't break that pattern.

## Commands

```
pytest                    # run all tests
pytest tests/test_X.py    # run a single file
pytest -x                 # stop on first failure
ruff check .              # lint
ruff check --fix .        # lint + auto-fix
ruff format .             # format
```

## Module Map

| Path | Responsibility |
|------|----------------|
| `aingram/store.py` | `MemoryStore` — the only public API for reads and writes. All business logic lives here. |
| `aingram/storage/engine.py` | `StorageEngine` — low-level SQLite CRUD and threading lock. No business logic. |
| `aingram/storage/schema.py` | DDL strings, `SCHEMA_VERSION`, `apply_schema()`, migration functions (`_migrate_vN_to_vM`). |
| `aingram/storage/queries.py` | Shared query utilities (reciprocal rank fusion, etc.). |
| `aingram/trust/` | Ed25519 signing, verification, content hashing, RFC-8785 canonicalization. |
| `aingram/trust/session.py` | `SessionManager` — per-session Ed25519 keypair and sequence counter. |
| `aingram/processing/` | `NomicEmbedder` (ONNX embeddings), `GlinerExtractor` (GLiNER2 multitask-large NER), `protocols.py` (`ContradictionClassifier`, `EntityExtractor`, `LLMProcessor` protocols). |
| `aingram/models/` | `ModelManager` — ONNX model file download and caching. |
| `aingram/extraction/` | Entity/relationship extraction (local LLM via Ollama, or Anthropic Sonnet). |
| `aingram/consolidation/` | Memory consolidation: decay, merge, contradiction detection, knowledge synthesis. |
| `aingram/consolidation/deberta.py` | `DeBERTaContradictionClassifier` — lazy-loaded DeBERTa-v3 NLI ONNX model for local contradiction detection. Implements `ContradictionClassifier` protocol. |
| `aingram/consolidation/contradiction.py` | `ContradictionDetector` orchestrator (entity grouping, pair iteration, recency fallback). `LLMContradictionClassifier` wraps `LLMProcessor` as a classifier. |
| `aingram/graph/` | Knowledge graph traversal for graph-augmented recall. |
| `aingram/integrations/` | Thin adapters: LangChain, CrewAI, LangGraph, AutoGen, smolagents. |
| `aingram/viz/` | Local HTTP visualization server (`aingram viz`). |
| `aingram/watch.py` | `watch_loop()` — live tail of new memory entries. |
| `aingram/cli.py` | Typer CLI entry point — all `aingram` subcommands. |
| `aingram/worker.py` | Background task queue processor (entity extraction jobs). |
| `aingram/config.py` | `AIngramConfig` — layered config (kwargs > env > TOML > defaults). |
| `aingram/mcp_server.py` | MCP server for Claude/Cursor tool integration. |
| `aingram/capture/` | Opt-in capture daemon for AI coding tools. Separate SQLite queue, per-tool adapters, filter pipeline, drain thread. |
| `aingram/capture/daemon.py` | Starlette HTTP app, `create_app()` factory, `run_daemon()` entry point with uvicorn. |
| `aingram/capture/queue.py` | `CaptureQueue` — thread-safe SQLite WAL wrapper for `capture_queue.db`. |
| `aingram/capture/drain.py` | `CaptureDrain` — background thread that dequeues records and calls `MemoryStore.remember()`. Triggers `store.consolidate()` automatically every `consolidation_interval_records` successfully drained records. |
| `aingram/capture/adapters/` | Per-tool adapters: Claude Code, Cursor, Gemini, Aider, Copilot, Cline, ChatGPT. |
| `aingram/capture/filters.py` | `apply_filters()` — `@nocapture` opt-out, secret redaction, container tag resolution. |
| `aingram/capture/config.py` | `CaptureConfig`, `ToolConfig`, `AiderToolConfig`, `ChatGPTToolConfig`. |
| `aingram/types.py` | All public dataclasses and enums. |
| `aingram/exceptions.py` | Exception hierarchy. |

## Architecture Invariants

1. **`MemoryStore` is the only public write path.** Never call `engine.store_entry()` or `_conn.execute()` directly from outside the `storage/` package. All business logic — signing, embedding, FTS indexing, dual-write — lives in `MemoryStore.remember()`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bozbuilds/AIngram](https://github.com/bozbuilds/AIngram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
