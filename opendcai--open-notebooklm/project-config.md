---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Open-NotebookLM — an open-source knowledge notebook platform (inspired by Google NotebookLM). Full-stack: FastAPI backend + React/TypeScript frontend + LangChain/LangGraph workflow engine. Supports knowledge base management, AI chat, podcast generation, mindmaps, flashcards, quizzes, PPT generation, deep research, and data extraction (SQLBot).

## Commands

```bash
# Backend (port 8213, hot-reload)
uvicorn fastapi_app.main:app --host 0.0.0.0 --port 8213 --reload

# Frontend (port 3001, Chinese version is primary)
cd frontend_zh && npm run dev -- --port 3001 --host 0.0.0.0

# Full stack (backend + frontend + monitor)
./scripts/start.sh
./scripts/stop.sh

# Backend with local GPU services
./scripts/start_backend.sh --tts-gpu 6 --embedding-gpu 7 --mineru-gpu 6

# Frontend build
cd frontend_zh && npm run build   # tsc && vite build

# Python env
conda activate szl-dev
```

No formal test suite. Ad-hoc test scripts exist in `fastapi_app/tests/` and `scripts/test_*.py`.

## Architecture

### Three-Layer Stack

```
frontend_zh/          React 18 + Vite + Tailwind + Zustand
    ↓ /api/v1/*       (Vite proxy → localhost:8213)
fastapi_app/          FastAPI + Pydantic, routers → services → providers
    ↓                 calls workflow engine for heavy AI tasks
workflow_engine/      LangChain + LangGraph, state-machine workflows
```

### Backend (`fastapi_app/`)

Entry: `main.py` → `create_app()`. All routers under `/api/v1`.

Key routers:
- `kb.py` — mega-router: sources, chat, QA, podcast, mindmap, flashcards, quiz, deep research
- `kb_workspace.py` — ThinkFlow workspace (documents, notes, outputs)
- `kb_notebooks.py`, `kb_sources.py` — CRUD
- `data_extract.py` — SQLBot data extraction
- `paper2ppt.py`, `paper2drawio.py` — document conversion

Middleware stack (order matters): CORS → LoggingMiddleware → APIKeyMiddleware (hardcoded key `df-internal-2024-workflow-key`).

Config: `config/settings.py` → `AppSettings(BaseSettings)`, reads from `fastapi_app/.env` then `.env.local` (override).

### Provider Pattern (`fastapi_app/providers/`)

ABC base classes in `base.py`: `EmbeddingProvider`, `TTSProvider`, `SearchProvider`. Concrete implementations auto-selected by env vars (e.g., `SEARCH_PROVIDER=serper|serpapi|bocha`).

### Workspace Storage (`fastapi_app/services/workspace_repository.py`)

JSON-file-based storage (no database for workspace data). `WorkspaceStorageMixin` provides manifest read/write. Services inherit this mixin: `document_service.py`, `thinkflow_workspace_service.py`, `output_v2_service.py`.

Legacy directory migration handled by `ensure_workspace_migrated()` — moves `documents/`, `workspace_items/`, `outputs_v2/` into unified `workspace/` dir.

### Path Management (`fastapi_app/notebook_paths.py`)

All file paths go through `NotebookPaths`. Directory layout:
```
outputs/{user_id}/{safe_title}_{notebook_id}/
├── sources/{source_stem}/original|mineru|markdown/
├── vector_store/
├── workspace/documents|notes|outputs/
├── ppt|mindmap|podcast|drawio/{timestamp}/
```

### Workflow Engine (`workflow_engine/`)

LangGraph-based state-machine workflows:
- `workflow/registry.py` — `@register` decorator for workflow registration
- `workflow/__init__.py` — auto-discovers `wf_*.py` files, provides `run_workflow(name, state)`
- `state.py` — dataclass-based states: `MainState`, `DFState`, `KBPodcastState`, `KBMindMapState`, etc.
- `agentroles/cores/base_agent.py` — `BaseAgent` ABC with execution modes (simple, ReAct, parallel, graph)
- `graphbuilder/graph_builder.py` — `GenericGraphBuilder` for LangGraph state graphs
- `promptstemplates/` — Jinja2-based prompt templates

Active workflows: `wf_kb_page_content`, `wf_kb_podcast`, `wf_kb_mindmap`, `wf_paper2ppt_parallel_consistent_style`, `wf_intelligent_qa`.

### Frontend (`frontend_zh/`)

React 18 + TypeScript + Zustand + Tailwind CSS + Vite.

- `src/App.tsx` — root: Dashboard or NotebookView, optional Supabase auth
- `src/config/api.ts` — `apiFetch()` wrapper with API key header
- `src/stores/authStore.ts` — Zustand auth store
- `src/components/thinkflow/` — ThinkFlow workspace (active development on `thinkflow` branch)
- `src/components/notes/` — block editor (Notion-like)
- `src/components/ui/` — shared UI primitives

Design system: warm neutrals + Electric Coral accent (#F43F5E), Newsreader serif headlines, Inter body.

### Source Processing (`fastapi_app/source_manager.py`)

`SourceManager` handles file import: PDF (via MinerU), DOCX, PPTX, MD, URL → markdown. Integrates MinerU and SAM3 for document parsing.

### Deep Research (`fastapi_app/modules/deep_research/`)

ReAct agent with tools: web search, page visit, file operations, Python execution, Google Scholar.

### SQLBot (`fastapi_app/modules/agents/`)

LangGraph-based pipeline: understanding → retrieval → generation → validation → EGA (Extensional Graph Analytics). RAG with BM25 + hybrid retriever.

## Key Conventions

- Env loading: `.env` first, `.env.local` overrides (both in `fastapi_app/`)
- Auth: optional Supabase JWT, degrades to guest mode when unconfigured

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenDCAI/Open-NotebookLM](https://github.com/OpenDCAI/Open-NotebookLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
