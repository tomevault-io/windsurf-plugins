---
trigger: always_on
description: <!-- This file is for AI agents (Claude Code, Cursor, Copilot, etc.) working in this repository.
---

# CLAUDE.md

<!-- This file is for AI agents (Claude Code, Cursor, Copilot, etc.) working in this repository.
     It also serves as a shared project memory — recording conventions, architecture decisions,
     and common patterns that all contributors (human or AI) should follow.
     Symlinked as AGENT.md and MEMORY.md for compatibility with other tools. -->

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
# Install in development mode
uv sync --all-extras

# Run all tests (use python -m pytest to avoid system pytest conflicts)
uv run python -m pytest

# Run a single test file
uv run python -m pytest tests/test_chunker.py

# Run a specific test
uv run python -m pytest tests/test_store.py::test_upsert_and_search -v

# Serve docs locally
uv run mkdocs serve

# Run the CLI
uv run memsearch --help
```

## Architecture

**memsearch** is a semantic memory search engine for markdown knowledge bases, built on Milvus.

### Data Flow

```
Markdown files → Scanner → Chunker → Embedder → MilvusStore
                                                      ↓
                               User query → Embedder → Hybrid Search (dense + BM25 + RRF) → Results
```

### Core Library (`src/memsearch/`)

- **`core.py`** — `MemSearch` class: the public Python API that orchestrates everything. Entry point for `index()`, `search()`, `compact()`, `watch()`.
- **`store.py`** — `MilvusStore`: Milvus wrapper handling collection creation, upsert, hybrid search (dense cosine + BM25 sparse + RRF reranking), and cleanup. The `chunk_hash` (composite ID of source+lines+content+model) is the VARCHAR primary key.
- **`chunker.py`** — Splits markdown by headings into `Chunk` dataclasses. SHA-256 content hash enables dedup. `compute_chunk_id()` generates composite IDs matching OpenClaw's format.
- **`embeddings/__init__.py`** — `EmbeddingProvider` protocol + lazy-loading factory (`get_provider()`). Providers: openai (default), google, voyage, jina, mistral, ollama, local, onnx.
- **`scanner.py`** — Walks directories to find `.md`/`.markdown` files, returns `ScannedFile` list.
- **`config.py`** — Layered TOML config: dataclass defaults → `~/.memsearch/config.toml` → `.memsearch.toml` → CLI flags.
- **`cli.py`** — Click CLI wrapping the Python API. All commands resolve config via `resolve_config()` then instantiate `MemSearch`.
- **`watcher.py`** — `watchdog`-based file watcher with debounce, used by `memsearch watch` and the Claude Code plugin.
- **`compact.py`** — LLM-powered chunk summarization (OpenAI/Anthropic/Gemini).
- **`reranker.py`** — Optional cross-encoder reranking (ONNX or PyTorch backend). Disabled by default; enable via `reranker.model` config.

### Claude Code Plugin (`plugins/claude-code/`)

The plugin is a first-class component of memsearch — it's the primary real-world application that demonstrates the library in action. It gives Claude Code automatic persistent memory across sessions with zero user intervention.

**Architecture: 4 shell hooks + 1 skill + 1 background watcher**

```
plugins/claude-code/
├── hooks/
│   ├── common.sh                # Shared setup: PATH, memsearch detection, collection name, watch PID
│   ├── session-start.sh         # SessionStart: start watch, write session heading, inject recent memories
│   ├── user-prompt-submit.sh    # UserPromptSubmit: lightweight hint reminding Claude about memory skill
│   ├── stop.sh                  # Stop: extract last turn → haiku summarize (third-person) → append to daily .md (async)
│   ├── session-end.sh           # SessionEnd: stop watch process
│   └── parse-transcript.sh      # Last-turn extractor: finds last user question → EOF, formats with role labels for LLM (Python 3, no jq)
├── scripts/
│   └── derive-collection.sh     # Derive per-project collection name from project path
├── transcript.py                # JSONL transcript parser for Claude Code conversation files (L3 deep drill)
└── skills/
    └── memory-recall/
        └── SKILL.md             # Skill (context: fork): search → expand → transcript in subagent
```

**Key design: skill-based memory recall.** Memory retrieval is handled by a `memory-recall` skill that runs in a forked subagent context (`context: fork`). Claude automatically invokes the skill when it judges the user's question could benefit from historical context. The subagent autonomously performs search, evaluates relevance, expands promising results, and returns a curated summary — all without polluting the main conversation context.

**Three-layer progressive disclosure (all in subagent):**
1. **L1 (search):** Subagent runs `memsearch search` to find relevant chunks
2. **L2 (expand):** Subagent runs `memsearch expand <chunk_hash>` to get full markdown sections
3. **L3 (transcript):** Subagent runs `python3 ${CLAUDE_PLUGIN_ROOT}/transcript.py <jsonl>` to drill into original conversations

**Supporting hooks:**
- `SessionStart` injects cold-start context (recent daily logs) so Claude knows history exists
- `UserPromptSubmit` returns a lightweight `systemMessage` hint ("[memsearch] Memory available") to increase skill trigger awareness

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zilliztech/memsearch](https://github.com/zilliztech/memsearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
