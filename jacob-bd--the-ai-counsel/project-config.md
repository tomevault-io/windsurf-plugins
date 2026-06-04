---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

The AI Counsel is a 3-stage deliberation system where multiple LLMs collaboratively answer user questions through:
1. **Stage 1**: Individual model responses (with optional web search context)
2. **Stage 2**: Anonymous peer review/ranking to prevent bias
3. **Stage 3**: Chairman synthesis of collective wisdom

**Key Innovation**: Hybrid architecture supporting OpenRouter (cloud), Ollama (local), Groq (fast inference), direct provider connections, and custom OpenAI-compatible endpoints.

## Running the Application

**Quick Start:**
```bash
./start.sh
```

**Manual Start:**
```bash
# Backend (from project root)
uv run python -m backend.main

# Frontend (in new terminal)
cd frontend
npm run dev
```

**Ports:**
- Backend: `http://localhost:8001` (NOT 8000 - avoid conflicts)
- Frontend: `http://localhost:5173`
- MCP Server (SSE): Built-in at `/mcp` on the backend (`http://localhost:8001/mcp/sse`) — **10 action-based tools** (`council_deliberate`, `model_chat`, `advisor_debate`, `run_iterative_debate`, `council_settings`, `advisor_settings`, `personas`, `conversations`, `providers`, `config_backup`). See [`docs/mcp/TOOLS.md`](docs/mcp/TOOLS.md). `GET /api/health` reports `"mcp": {"tools": 10}`.

**Network Access:**
```bash
# Backend with network access:
LLM_COUNCIL_BIND_HOST=0.0.0.0 uv run python -m backend.main

# Frontend with network access:
cd frontend && npm run dev -- --host
```

**Backend bind variables:**
- `LLM_COUNCIL_BIND_HOST`: dev launcher bind host, default `127.0.0.1`. Use `0.0.0.0` only when you intentionally want LAN access.
- `LLM_COUNCIL_BIND_PORT`: dev launcher bind port, default `8001`.
- `LLM_COUNCIL_ADMIN_TOKEN`: required for remote access to `/api/settings/export`, `/api/settings/import`, and `/api/settings/reset`. Without it, those admin endpoints only accept direct loopback clients and reject proxied external clients.

**Installing Dependencies:**
```bash
# Backend
uv sync

# Frontend
npm install --prefix frontend
```

**Important**: If switching between Intel/Apple Silicon Macs with iCloud sync:
```bash
rm -rf frontend/node_modules && npm install --prefix frontend
```
This fixes binary incompatibilities (e.g., `@rollup/rollup-darwin-*` variants).

## Architecture Overview

### Backend (`backend/`)

**Provider System** (`backend/providers/`)
- **Base**: `base.py` - Abstract interface for all LLM providers
- **Implementations**: `openrouter.py`, `ollama.py`, `groq.py`, `openai.py`, `anthropic.py`, `google.py`, `mistral.py`, `deepseek.py`, `custom_openai.py`, `opencode.py` (OpenCode Zen + Go, chat/completions only)
- **Auto-routing**: Model IDs with prefix (e.g., `openai:gpt-4.1`, `ollama:llama3`, `custom:model-name`) route to correct provider
- **Routing logic**: `council.py:get_provider_for_model()` handles prefix parsing

**Core Modules**

| Module | Purpose |
|--------|---------|
| `council.py` | Orchestration: stage1/2/3 collection, rankings, title generation |
| `search.py` | Web search: DuckDuckGo, Tavily, Brave, Serper, TinyFish with Jina Reader content fetch |
| `settings.py` | Config management, persisted to `data/settings.json` |
| `config.py` | OpenRouter endpoint URL, data dir constant, settings-aware getters (`get_openrouter_api_key`, `get_council_models`, `get_chairman_model`, ...) that bridge env vars and `settings.py` |
| `costs.py` | Usage normalization, pricing lookup/cache, per-call cost attribution, and run-level cost reports |
| `prompts.py` | Default system prompts for all stages (Stage 1/2/3, Title, Query) |
| `main.py` | FastAPI app with streaming SSE endpoints, live progress tracking (`_active_runs`), and MCP server mount |
| `storage.py` | Conversation persistence in `data/conversations/{id}.json`; index entries include optional `run_summary`, `total_cost`, `cost_status`, `total_calls` via `derive_run_summary()` / `derive_conversation_cost()` |

### Frontend (`frontend/src/`)

| Component | Purpose |
|-----------|---------|
| `App.jsx` | Main orchestration, SSE streaming, conversation state |
| `ChatInterface.jsx` | User input, web search toggle, execution mode |
| `Stage1.jsx` | Tab view of individual model responses |
| `Stage2.jsx` | Peer rankings with de-anonymization, aggregate scores |
| `Stage3.jsx` | Chairman synthesis (final answer) |
| `CostReport.jsx` | Compact run-cost panel with total cost, token/call counts, confidence status, and per-model breakdown (uses `formatCost.js`) |
| `CouncilGrid.jsx` | Visual grid of council members with provider icons |
| `CouncilSetup.jsx` | Inline council editor on welcome screen (members, chairman, presets; auto-save) |
| `Settings.jsx` | 8-section settings: General, LLM API Keys, Council Config, Council Debate Config, Council System Prompts, Advisor System Prompts, Search Providers, Backup & Reset |
| `GeneralSettings.jsx` | Date format and response language (General section) |
| `Sidebar.jsx` | Conversation list with stacked date/time, run summaries, cumulative cost pill, sidebar search on summary text, inline delete confirmation |
| `SearchableModelSelect.jsx` | Searchable dropdown for model selection |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jacob-bd/the-ai-counsel](https://github.com/jacob-bd/the-ai-counsel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
