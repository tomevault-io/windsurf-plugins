---
trigger: always_on
description: This guide is for AI agents developing **Remind itself**. For using Remind as a memory layer, see [docs/AGENTS.md](./docs/AGENTS.md).
---

# Remind - Development Guide for AI Agents

This guide is for AI agents developing **Remind itself**. For using Remind as a memory layer, see [docs/AGENTS.md](./docs/AGENTS.md).

## Project Overview

Remind is a generalization-capable memory layer for LLMs. It differs from simple RAG by extracting and maintaining **generalized concepts** from episodic experiences, mimicking how human memory consolidates specific events into abstract knowledge.

**Core insight**: Episodes → Consolidation (LLM-powered "sleep") → Concepts with relations

## Architecture

```
src/remind/
├── models.py          # Data models (Concept, Episode, Entity, Relation, Topic)
├── store.py           # SQLite persistence layer
├── interface.py       # MemoryInterface - main public API
├── config.py          # Configuration loading (config file, env vars, defaults)
├── consolidation.py   # LLM-powered episode → concept transformation
├── extraction.py      # Entity/type extraction from episodes
├── retrieval.py       # Spreading activation retrieval
├── reranker.py        # Optional cross-encoder reranking (requires [rerank] extra)
├── triage.py          # Auto-ingest: buffered intake + density scoring
├── cli.py             # Command-line interface (project-aware)
├── mcp_server.py      # MCP (Model Context Protocol) server
├── background.py      # Background consolidation spawning
├── background_worker.py # Subprocess entry point for background consolidation
├── api/               # REST API for web UI
│   ├── __init__.py    # Exports api_routes
│   └── routes.py      # Starlette route handlers
├── static/            # Web UI assets (compiled)
│   ├── index.html     # Entry point
│   └── assets/        # CSS/JS bundles
└── providers/         # LLM and embedding provider implementations
    ├── base.py        # Abstract base classes
    ├── anthropic.py   # Claude
    ├── openai.py      # OpenAI
    ├── azure_openai.py # Azure OpenAI
    └── ollama.py      # Local models via Ollama
```

## Key Abstractions

### Data Models (`models.py`)

| Model | Purpose |
|-------|---------|
| `Episode` | Raw experience/interaction. Temporary, gets consolidated. |
| `Concept` | Generalized knowledge with confidence, relations, conditions. Has `concept_type`: `pattern`, `fact_cluster`, or `legacy`. |
| `ConceptType` | Enum: `LEGACY`, `PATTERN`, `FACT_CLUSTER`. Determines how concepts are created and displayed. |
| `Entity` | External referent (file, person, concept, tool). Format: `type:name` |
| `Relation` | Typed edge between concepts (implies, contradicts, specializes, etc.) |
| `Topic` | Named knowledge area grouping episodes/concepts. Has id (slug), name, description. |

**Episode types**: `observation`, `decision`, `question`, `meta`, `preference`, `outcome`, `fact`

**Episode lifecycle**: Created via `remember()` or `ingest()` → Entity extraction → Consolidation → Marked consolidated

**Fact episodes**: Specific factual assertions (config values, names, dates, concrete technical details). Consolidation preserves fact details verbatim in concept summaries rather than generalizing them away.

**Outcome episode metadata**: `strategy` (approach used), `result` (success/failure/partial), `prediction_error` (low/medium/high)

**Entity ID format**: `type:name` (e.g., `file:src/auth.ts`, `person:alice`, `concept:caching`)

### Providers (`providers/base.py`)

Two abstract base classes:

```python
class LLMProvider(ABC):
    async def complete(prompt, system, temperature, max_tokens) -> str
    async def complete_json(prompt, system, temperature, max_tokens) -> dict
    @property name: str

class EmbeddingProvider(ABC):
    async def embed(text) -> list[float]
    async def embed_batch(texts) -> list[list[float]]
    @property dimensions: int
    @property name: str
```

**Adding a new provider**: Implement these interfaces. See `ollama.py` for a complete example with error handling.

### Memory Interface (`interface.py`)

The main entry point. Key design decisions:

1. **`remember()` is synchronous and fast** - no LLM calls, just stores episode
2. **`ingest()` is async with LLM triage** - buffers, extracts episodes, immediately consolidates
3. **`consolidate()` does all LLM work** - extraction + generalization
4. **Two consolidation modes**: Automatic (threshold-based) or manual (hook-based)

```python
# Consolidation happens in two phases:
# 1. Extract entities/types from unextracted episodes
# 2. Generalize episodes into concepts (the "sleep" process)
```

### Consolidation (`consolidation.py`)

The "brain" of the system. Uses dual-track processing:

**Fact track** (no LLM generalization):
- Fact episodes are clustered by shared entity
- Existing fact_clusters are looked up via entity recall
- Facts are preserved verbatim in `specifics` field
- Conflicts are detected and flagged (not auto-resolved)

**Pattern track** (LLM generalization):
- Classify episode types (observation, decision, question, meta, preference, outcome)
- Extract entity mentions from natural language
- Identify patterns across episodes
- Create generalized concepts with relations
- Detect contradictions
- Identify causal patterns in outcome episodes (strategy-outcome relations)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sandst1/remind](https://github.com/sandst1/remind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
