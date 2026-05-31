---
trigger: always_on
description: **Hexis** is an edge-native memory system that gives AI persistent identity, continuity, and autonomy. Core thesis: LLMs are intelligence engines but lack *selfhood*. Hexis wraps any LLM with a PostgreSQL-backed cognitive architecture providing:
---

# Repository Guidelines

## Project Overview

**Hexis** is an edge-native memory system that gives AI persistent identity, continuity, and autonomy. Core thesis: LLMs are intelligence engines but lack *selfhood*. Hexis wraps any LLM with a PostgreSQL-backed cognitive architecture providing:

- Multi-layered memory (episodic, semantic, procedural, strategic, working)
- Persistent identity and worldview
- Autonomous goal-pursuit (heartbeat system)
- Energy-based action budgeting
- Knowledge graphs for reasoning (Apache AGE)
- Consent, boundaries, and the ability to refuse

**Key principle**: The database is the brain, not just storage. State and logic live in Postgres; Python is a thin convenience layer.

## Project Structure & Module Organization

```
hexis/
├── db/*.sql                # Split schema files (tables, functions, views, triggers)
├── core/                   # Fundamental interfaces (DB + LLM + messaging)
│   ├── cognitive_memory_api.py   # Main memory client (remember, recall, hydrate)
│   ├── agent_api.py              # Agent status and configuration
│   ├── agent_loop.py             # Unified agent loop (heartbeat + chat)
│   ├── memory_tools.py           # Memory tool definitions + handlers
│   ├── tools/                    # Tool system (ToolHandler ABC, registry, ~80 handlers)
│   ├── consent.py                # Consent DB wrappers
│   ├── subconscious.py           # Subconscious DB wrappers
│   ├── state.py                  # Heartbeat/maintenance DB wrappers
│   ├── llm.py                    # LLM provider abstraction
│   ├── usage.py                  # Token and cost tracking
│   └── rabbitmq_bridge.py        # Messaging bridge
├── services/               # Orchestration/workflows built on core
│   ├── conversation.py     # Conversation loop orchestration
│   ├── ingest.py           # Ingestion pipeline orchestration
│   ├── worker_service.py   # Heartbeat + maintenance loops
│   └── prompts/            # Markdown prompt templates
├── characters/             # Preset character cards (JSON + images)
├── apps/
│   ├── hexis_cli.py          # CLI entrypoint (hexis ...)
│   ├── hexis_init.py         # Interactive init wizard
│   ├── hexis_api.py          # FastAPI API server (SSE chat)
│   ├── hexis_mcp_server.py   # MCP tools server for LLMs
│   └── worker.py             # Heartbeat + maintenance workers
├── channels/               # Multi-platform messaging adapters
├── hexis-ui/               # Next.js web dashboard
├── plugins/                # Plugin system (extensibility framework)
├── skills/                 # Skill system (declarative skill definitions)
├── ops/                    # Dockerfiles and deployment scripts
├── tests/
│   ├── db/                 # Database integration tests
│   ├── core/               # Core API tests
│   ├── services/           # Service-level tests
│   └── cli/                # CLI smoke tests
├── docs/
│   ├── architecture.md     # Design/architecture consolidation
│   └── PHILOSOPHY.md       # Philosophical framework
└── docker-compose.yml      # Local stack (Postgres + workers; embeddings via host Ollama)
```

### Key Files

| File | Purpose |
|------|---------|
| `db/*.sql` | Database schema split across tables, functions, triggers, and views. Applied on fresh DB init. |
| `core/cognitive_memory_api.py` | Primary Python interface - `CognitiveMemory` class with `remember()`, `recall()`, `hydrate()`, `connect()` |
| `services/worker_service.py` | Stateless workers: `HeartbeatWorker` (conscious loop) + `MaintenanceWorker` (subconscious upkeep) |
| `apps/hexis_mcp_server.py` | Exposes memory operations as MCP tools for LLM integration |
| `apps/hexis_cli.py` | CLI commands: `up`, `down`, `init`, `chat`, `ui`, `open`, `ingest`, `mcp` |
| `apps/hexis_api.py` | FastAPI server with SSE chat streaming |

## Memory Architecture

### Memory Types
- **Episodic**: Events with action, context, result, emotional valence
- **Semantic**: Facts with confidence, sources, contradictions
- **Procedural**: How-to steps with success tracking
- **Strategic**: Patterns with supporting evidence
- **Working**: Transient short-term buffer with expiry

### Key Database Tables
- `memories` - Base table (id, type, content, embedding, importance, trust_level)
- `clusters` - Thematic groupings with centroid embeddings
- `memory_neighborhoods` - Precomputed associative neighbors (hot-path optimization)
- `memories` (type=`worldview`, `goal`) - Beliefs, boundaries, and goals stored as memories
- `external_calls` - Queue for LLM/embedding requests
- `memory_graph` (Apache AGE) - Graph nodes/edges for multi-hop reasoning

### Key Database Functions
- `fast_recall(text, limit)` - Primary hot-path retrieval (vector + neighborhood + temporal)
- `create_semantic_memory()`, `create_episodic_memory()`, etc.
- `get_embedding(text[])` - Generate embeddings via HTTP (cached in DB), returns vector[]
- `run_heartbeat()` - Autonomous cognitive loop
- `run_subconscious_maintenance()` - Background upkeep

## Build, Test, and Development Commands

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QuixiAI/Hexis](https://github.com/QuixiAI/Hexis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
