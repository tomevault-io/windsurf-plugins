---
trigger: always_on
description: > Agentic coding guide for auto-scholar: FastAPI + LangGraph backend with Next.js 16 frontend.
---

# AGENTS.md — auto-scholar

> Agentic coding guide for auto-scholar: FastAPI + LangGraph backend with Next.js 16 frontend.

**Generated:** 2026-02-26 | **Commit:** 129b58d | **Branch:** main

## Quick Commands

```bash
# Backend
uv sync --extra dev                        # Install deps
find backend -name '*.py' -exec python -m py_compile {} +  # Compile check all
python -m py_compile backend/schemas.py      # Compile check single file
ruff check backend/                          # Lint all
ruff check backend/main.py                   # Lint single file
ruff format backend/ --check                 # Check formatting
ruff format backend/                         # Auto-format

# Backend tests (pytest)
uv run pytest tests/ -v                             # Run all tests
uv run pytest tests/test_integration.py -v          # Run single file
uv run pytest tests/test_integration.py::test_full_workflow -v  # Run single test
uv run pytest tests/test_exporter.py::test_export_markdown -v   # Another example
uv run pytest -x                                    # Stop on first failure
uv run pytest -m "not slow"                         # Skip slow tests
uv run pytest -m "not integration"                  # Skip integration tests
uv run pytest --cov=backend tests/                  # With coverage

# Frontend
cd frontend && bun install                   # Install deps
cd frontend && bun run build                 # Production build
cd frontend && bun x tsc --noEmit            # Type check
cd frontend && bun run lint                  # ESLint

# Frontend tests (vitest + playwright)
cd frontend && bun test                      # Run unit tests (vitest)
cd frontend && bun test src/__tests__/store.test.ts  # Single test file
cd frontend && bun run test:e2e              # Run E2E tests (playwright)

# Docker
docker compose up --build                    # Full stack (backend:8000 + frontend:3000)

# DO NOT run these from agents (long-running):
# uvicorn backend.main:app --reload --port 8000
# cd frontend && bun run dev
```

## Project Structure

```
auto-scholar/
├── backend/                    # FastAPI + LangGraph backend
│   ├── main.py            # REST endpoints (start, stream, approve, status, export, sessions)
│   ├── workflow.py        # LangGraph graph + QA retry router + reflection loop
│   ├── nodes.py           # 6 agent nodes (plan, search, extract, draft, QA, reflection) — 948 lines, largest file
│   ├── state.py           # AgentState TypedDict with Annotated reducers
│   ├── schemas.py         # Pydantic V2 models — single source of truth for 40+ types
│   ├── constants.py       # Config constants with trade-off rationale comments
│   ├── prompts.py         # Centralized LLM prompt templates
│   ├── config/
│   │   └── loader.py      # YAML model config loader with ${ENV_VAR:-default} substitution
│   ├── llm/
│   │   ├── router.py      # Task-aware model selection (scoring by reasoning/creativity/latency)
│   │   └── task_types.py  # TaskType enum + per-task requirements
│   ├── evaluation/        # 7-dimension evaluation framework (see backend/evaluation/AGENTS.md)
│   └── utils/
│       ├── llm_client.py  # AsyncOpenAI wrapper + multi-provider fallback chains
│       ├── scholar_api.py # Semantic Scholar + arXiv + PubMed clients (590 lines)
│       ├── event_queue.py # SSE debouncing engine (200ms window + semantic boundaries)
│       ├── exporter.py    # Markdown/DOCX export with 4 citation styles
│       ├── citations.py   # {cite:N} → [N] normalization
│       ├── claim_verifier.py  # Batch claim extraction + entailment verification
│       ├── source_tracker.py  # Circuit breaker for failed sources (3 fails = skip 2min)
│       ├── http_pool.py   # Connection pooling (limit=50, TTL=300s)
│       ├── fulltext_api.py    # Unpaywall + OpenAlex PDF URL resolution
│       ├── charts.py      # Matplotlib chart generation
│       └── logging.py     # JSON structured logging with thread_id context
├── config/
│   └── models.yaml        # Multi-model config (OpenAI, DeepSeek, Ollama) with capability scores
├── frontend/              # Next.js 16 + React 19
│   └── src/
│       ├── app/           # App router (page.tsx — 320 lines, layout.tsx)
│       ├── components/    # UI components (console/, workspace/, approval/, ui/)
│       ├── store/         # Zustand state (research.ts — 363 lines)
│       ├── lib/api/       # API client with SSE + REST
│       ├── i18n/          # Internationalization (en/zh) via next-intl
│       ├── types/         # TypeScript types mirroring backend schemas 1:1
│       └── __tests__/     # Vitest unit tests
├── tests/                 # Backend pytest tests (34 files, ~7900 lines)
├── .github/workflows/ci.yml  # 3 parallel jobs: lint, test (matrix 3.11/3.12), frontend
└── pyproject.toml         # Python >=3.11, pytest asyncio_mode=auto
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add/modify workflow node | `backend/nodes.py` | 6 agents, uses prompts from `prompts.py` |
| Change workflow graph | `backend/workflow.py` | LangGraph StateGraph, `_qa_router` for retry logic |
| Add API endpoint | `backend/main.py` | FastAPI, all routes under `/api/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CAICAIIs/Auto-Scholar](https://github.com/CAICAIIs/Auto-Scholar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
