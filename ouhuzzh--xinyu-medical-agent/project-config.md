---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```powershell
# Setup
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r requirements.txt
cd frontend && npm install && cd ..

# Start both backend + frontend
.\start_frontend_app.ps1 -Restart -SkipInstall

# Start backend only
.\venv\Scripts\python.exe project\api_app.py        # FastAPI on :8000

# Start frontend only
cd frontend && npm run dev                           # Vite on :5173

# Start Gradio admin console (separate from user app)
.\venv\Scripts\python.exe project\app.py             # Gradio on :7860
```

## Testing

```powershell
# Syntax check (fast)
.\venv\Scripts\python.exe -m compileall project tests

# Full regression
.\venv\Scripts\python.exe -m unittest discover -s tests -v

# Single test module
.\venv\Scripts\python.exe -m unittest tests.test_api_app -v

# Frontend build check
cd frontend && npm run build

# Split-app smoke (no live model needed)
.\scripts\smoke_split_app.ps1 -SkipChat

# Split-app smoke with live chat (requires model provider)
.\scripts\smoke_split_app.ps1
```

## Architecture

This is a medical assistant with RAG, appointment booking, and multi-turn memory. Two separate user surfaces share the same backend.

### Request Flow

```
React/Vite (:5173) → FastAPI (:8000) → ChatInterface → LangGraph graph
                                                           ↓
                                         ┌─────────────────┼──────────────────┐
                                         ▼                 ▼                  ▼
                                   Medical RAG      Appointment Skill   Memory/State
                                         ↓                 ↓                  ↓
                                   pgvector+tsvector  PostgreSQL         Redis+PostgreSQL
```

### Backend Entry Points

- `project/api_app.py` — uvicorn launcher for FastAPI (user app)
- `project/app.py` — Gradio admin/debug console
- Both import `project/config.py` which reads `project/.env` for all settings

### Key Modules

**`project/rag_agent/`** — LangGraph workflow (the core intelligence):
- `graph.py` — builds the StateGraph with 20+ nodes, compiled with a checkpointer; integrates skill-registered nodes dynamically
- `graph_state.py` — `State` (main graph) and `AgentState` (subgraph) TypedDicts; `skill_data: Dict[str, Any]` holds per-skill state
- `routing_nodes.py` — intent classification, turn analysis, department recommendation
- `rag_nodes.py` — query rewrite, retrieval orchestration, answer grounding, fallback
- `appointment_nodes.py` — booking/cancellation nodes invoked from the graph
- `edges.py` — conditional edge functions that route between nodes
- `prompts.py` — all LLM prompt templates
- `tools.py` — LangGraph tool definitions
- `node_helpers.py` — shared helper functions (rule-based intent detection, text sanitization)
- `persistent_checkpointer.py` — `PersistentInMemorySaver` that serializes checkpoints to a pkl file

**`project/skills/`** — pluggable skill framework for extensible intent routing:
- `base_skill.py` — `BaseSkill` ABC: each skill declares `name`, `priority`, `intent_label`, `match()`, `register_nodes()`, `register_edges()`, `get_route_targets()`, `get_state_schema()`
- `registry.py` — `SkillRegistry` singleton: `classify_intent()` tries skills in priority order; `register_all_nodes()`/`register_all_edges()` inject into the graph builder
- `greeting_skill.py` — proof-of-concept skill (priority 10), routes greetings to a dedicated handler → END
- `medical_rag_skill.py` — proof-of-concept skill (priority 60), routes medical questions to the existing `rewrite_query` chain
- Enabled via `SKILLS_ENABLED=true` in `.env`; when disabled, graph falls back to hardcoded routing

**`project/llm_tiered_router.py`** — tiered LLM routing with per-provider circuit breaker:
- Light tier (intent classification, query planning) vs strong tier (answer generation, department recommendation)
- Configured via `LLM_TIERS_JSON` env var; degrades to single-tier if unset
- `CircuitBreaker`: closed → open → half_open state machine with failure threshold and recovery timeout

**`project/core/`** — orchestration and infrastructure:
- `chat_interface.py` — `ChatInterface` facade that the API calls; wraps graph invocation, memory, SSE streaming, and document management
- `rag_system.py` — RAG pipeline: embedding, hybrid retrieval, reranking
- `document_manager.py` — document CRUD, chunking, vector storage
- `document_chunker.py` — text splitting with parent-child chunk strategy
- `knowledge_base_sync.py` — official-source sync with content-hash dedup and soft delete
- `medical_source_ingest.py` — MedlinePlus, NHC, WHO importers
- `qa_eval.py` — offline retrieval/answer/route quality evaluator for benchmarking
- `ablation.py` — `AblationStudy` framework: disables pipeline components (rewrite, hybrid, rerank) to measure their independent contribution

**`project/benchmarks/`** — offline evaluation and ablation studies:
- `run_ablation_study.py` — runs ablation across pipeline config variants
- `resume_benchmarks.py` — end-to-end benchmark with resume/summary reporting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ouhuzzh/xinyu-medical-agent](https://github.com/ouhuzzh/xinyu-medical-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
