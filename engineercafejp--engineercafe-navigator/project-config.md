---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Engineer Cafe Navigator — a multilingual voice AI agent system for Fukuoka Engineer Cafe. Monorepo with a Next.js frontend and a Python/LangGraph backend.

## Development Commands

### Frontend (Next.js) — runs from `frontend/`

```bash
cd frontend
pnpm dev              # Dev server at http://localhost:3000
pnpm build            # Production build
pnpm lint             # ESLint
pnpm typecheck        # TypeScript type checking (tsc --noEmit)
pnpm test             # Run test suite
pnpm test:e2e         # Playwright E2E tests
```

### Backend (FastAPI/LangGraph) — runs from `backend/`

```bash
cd backend
python -m uvicorn main:app --reload --host 0.0.0.0 --port 8000  # Dev server
pytest -m "not ragas and not slow" --tb=short -q                  # Unit tests (fast)
pytest                                                             # All tests
ruff check .                                                       # Linting
black --check .                                                    # Format check
black .                                                            # Auto-format
```

### Docker (full stack)

```bash
make dev              # docker-compose up (frontend:3000 + backend:8000)
make setup            # Initial setup (mise install + deps + Docker build)
make lint             # Lint both frontend and backend
make test:backend     # Backend tests excluding slow/ragas markers
```

### Specific Agent Testing

```bash
make test-agent AGENT=business_info QUERY='営業時間は？'
make debug-agent                    # Interactive agent debugger
```

## Architecture

### Monorepo Structure

```
frontend/          Next.js 15 (App Router) + React 19 + TypeScript
  src/app/         Pages and API routes
  src/app/api/     API route handlers (voice, slides, marp, qa, character, calendar, admin)
  src/lib/         Shared libraries (audio, memory, STT correction, lip-sync)
  e2e/             Playwright E2E tests

backend/           FastAPI + LangGraph + Python 3.11+
  main.py          FastAPI application entry point
  agents/          Agent implementations (12 agents)
  workflows/       LangGraph workflow definitions (main_workflow, reception_workflow)
  tools/           Shared tools (calendar_service, enhanced_rag, tavily_search)
  config/          Routing constants, prompt templates
  utils/           Input sanitizer, custom exceptions
  tests/           pytest test suite
  evaluation/      RAGAS evaluation scripts

supabase/          Database migrations and config
```

### AI Agent Architecture

The backend uses **LangGraph** with a Supervisor Pattern:

- **OrchestratorAgent** controls 12 specialized agents
- Agents: BusinessInfoAgent, FacilityAgent, EventAgent, GeneralKnowledgeAgent, CharacterControlAgent, SlideAgent, STTAgent, VoiceAgent, OCRAgent, FarewellAgent, plus agent_tools
- LLM: OpenRouter (Gemini) via LangChain
- RAG: EnhancedRAGSearch with Supabase RPC `search_knowledge_base()` + Tavily web search fallback
- Embeddings: OpenRouter API (`openai/text-embedding-3-small`, 1536 dimensions)

**Reception subgraph (Wave 7, PR #390)**: Reception handling is a first-class LangGraph subgraph. The three previously separate implementations (main_workflow.py inline handlers, reception_workflow.py standalone, api/reception.py) are unified. `reception_workflow.py` is invoked via `invoke_reception_subgraph()` with explicit state conversion functions. `api/reception.py` uses a singleton workflow instance (protected by `asyncio.Lock`). `consultation` routes to `general_knowledge` (not `business_info`).

**Multilingual RAG — tRAG pattern (Wave 7, PR #389)**: The knowledge base is Japanese-only. For English queries, tRAG translates to Japanese before embedding lookup and sets `language="ja"` on the RAG call. Chinese and Korean queries skip translation and use cross-lingual embeddings directly. `text_fallback_search` always filters by `"ja"`. Entity labels and advice templates support en/zh/ko. RAGAS targets: ja >= 0.85, en >= 0.75, zh/ko >= 0.65 (answer_correctness).

The frontend is a **pure UI layer** — all AI processing is proxied to the backend via `backendFetch()`.

### Key Data Flow

- Voice: Browser → `/api/voice` (FE proxy) → Backend STT/TTS → Browser
- Q&A: Browser → `/api/qa` (FE proxy) → Backend `/api/chat` → LangGraph → RAG/Web search → Response
- Reception: Browser → `/api/reception/*` (FE proxy) → Backend → `invoke_reception_subgraph()` → LangGraph reception subgraph → Response
- Calendar: Browser → `/api/calendar` (FE proxy) → Backend `/api/calendar` → Google Calendar ICS
- Slides: Marp markdown in `frontend/src/slides/` → `/api/marp` renders HTML → MarpViewer component

### Database

PostgreSQL (Supabase) with pgvector. Key tables:
- `knowledge_base`: RAG entries with 1536-dim embeddings (OpenRouter via text-embedding-3-small)
- `conversation_sessions` / `conversation_history`: Chat state
- `agent_memory`: Short-term memory with 3-minute TTL
- `reception_sessions` / `visits`: Reception flow state
- RLS enabled on all tables; use service role key for server-side access

### External Services


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EngineerCafeJP/engineercafe-navigator](https://github.com/EngineerCafeJP/engineercafe-navigator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
