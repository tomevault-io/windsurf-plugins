---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Laya

Laya is a local-first desktop app (Tauri + Svelte + Python) that intercepts professional tool events (Jira, Slack, Gmail, GitHub, Bitbucket, Linear, Notion, Google Calendar, Outlook Calendar, Outlook Email), classifies them with LLM-powered personas (Engineer, Comms, Ops, Sales, HR, Finance), stages actions, and presents Action Cards for user approval. n8n handles event ingestion and outbound action execution.

## Development Commands

```bash
# One-time setup (creates venvs, installs deps)
scripts/setup-dev.sh

# Run dev environment (Tauri dev server manages engine + n8n)
scripts/dev.sh

# Run engine standalone (for backend-only work)
cd engine && source .venv/bin/activate && python -m laya.main

# Run tests
cd engine && source .venv/bin/activate && pytest

# Run a single test file
cd engine && source .venv/bin/activate && pytest tests/test_cards_api.py

# Run a single test
cd engine && source .venv/bin/activate && pytest tests/test_cards_api.py::test_function_name -v

# Frontend dev only (no Tauri shell)
cd ui && npm run dev

# Type check frontend
cd ui && npx svelte-check

# Frontend unit tests (vitest — pure-logic modules like the feed reducer/flip utils)
cd ui && npm test

# Production build
scripts/build.sh
```

## Architecture

```
Event Sources → n8n (port 45678) → Engine (port 8420) → UI (Tauri + Svelte, port 5173)
                                         ↓
                               SQLite + ChromaDB (~/.laya/data/)
```

**Three layers:**
- **Engine** (`engine/laya/`): Python FastAPI backend. Pipeline processes events through `ingest → space_resolution → rules → router → workers → stager → emit` with post-emit async steps (see below). LLM calls go through LiteLLM (`llm/client.py`), which can also drive installed CLI coding agents (Claude Code, Codex, Gemini, Pi) as inference backends via `llm/agent_backend.py` (model-id form `agent/<id>/<model>`). ~27 API routers in `api/` — the large card surface is split into `cards_common` (shared row→model helpers) + route modules (`cards_feed`/`cards_lifecycle`/`cards_readstate`/`cards_payload`/`cards_agent`/`cards_groups`), aggregated in-place via `include_router` into one `cards_router` (order matters: `/cards/grouped` before `/cards/{card_id}`; P7-6). Pydantic models in `models/`. Async throughout (aiosqlite, httpx).
- **UI** (`ui/src/`): SvelteKit frontend using Svelte 5 runes (`$state`, `$derived`, `$effect`, `$props`). Skeleton UI v4 + Tailwind CSS v4. Static adapter (SPA mode). Key routes: feed, coherence, dashboard, settings, workspace, omni, setup, status, legal.
- **Tauri Shell** (`ui/src-tauri/`): Rust process that manages engine and n8n lifecycle (`sidecar.rs`, `n8n.rs`), tray icon, and native APIs.

**Pipeline flow** (`engine/laya/pipeline/`):

Main pipeline (`queue.py` orchestrates):
1. `ingest.py` — receives normalized events from n8n webhooks
2. `space_resolution.py` — resolves event to a space
3. `rules.py` — applies user-defined filter/routing rules
4. `router.py` — classifies event → persona with priority
5. `workers.py` — dispatches persona workers (Engineer/Comms/Ops/Sales/HR/Finance)
6. `stager.py` — stages actions via LLM
7. `emit.py` — creates Action Cards in SQLite

Post-emit (triggered by `emit.py`):
- `entity_resolution.py` — resolves semantic entities across platforms
- `context_grouping.py` — groups related cards into context groups
- `trace.py` — indexes into ChromaDB for semantic search
- `tags.py` — persists stager-suggested tags
- `group_summary.py` — rolling LLM summary per entity group
- `omni.py` — rolling cross-platform summary with progressive summarization

Supporting pipelines (triggered separately):
- `chat.py` — chat processing pipeline (hybrid retrieval, RRF, context packing, tool loop)
- `executor.py` — action execution service, delegates to egress and manages card lifecycle
- `context_presets.py` — context association strictness presets
- `learn.py` — extracts classification rules (priority/persona) from user corrections; `context_learn.py` — extracts **context rules** (natural-language grouping directives, distinct from classification rules) from link/unlink corrections. BOTH LLM-consolidate their learned rules once they exceed a threshold (capping unbounded router/grouping-prompt growth) and share their fetch/mark-processed/space-scan query helpers via `pipeline/learn_common.py`. Learned and manual context rules are managed via `api/context_rules_api.py` and injected into the context-grouping prompt
- `processing_rules.py` — applies automated processing rules, logging every firing to `processing_rule_firings`
- `briefing.py` — generates daily briefings
- `summarize.py` — daily summary generation
- `feedback.py` / `budget.py` — feedback processing and monthly $ token-budget tracking
- `agent_budget.py` — window-based usage-limit budgeting for agent inference backends (auto-pause ingestion when an agent's rolling window is exhausted, auto-resume at window reset)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aayushch/laya](https://github.com/aayushch/laya) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
