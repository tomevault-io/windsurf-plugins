---
trigger: always_on
description: AI Reader V2 is a Chinese novel analysis platform. Users upload TXT novels, the system splits them into chapters, optionally pre-scans for high-frequency entities (jieba + LLM classification), then uses an LLM to extract structured facts per chapter (ChapterFact), aggregating them into entity profiles, visualizations, and a Q&A system. Supports local Ollama and cloud OpenAI-compatible APIs. All data stays on the local machine.
---

# CLAUDE.md - AI Reader V2

## Project Overview

AI Reader V2 is a Chinese novel analysis platform. Users upload TXT novels, the system splits them into chapters, optionally pre-scans for high-frequency entities (jieba + LLM classification), then uses an LLM to extract structured facts per chapter (ChapterFact), aggregating them into entity profiles, visualizations, and a Q&A system. Supports local Ollama and cloud OpenAI-compatible APIs. All data stays on the local machine.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 19 + TypeScript 5.9 + Vite 7 |
| UI | Tailwind CSS 4 + shadcn/ui + Radix UI + Lucide icons |
| State | Zustand 5 (7 stores) |
| Routing | React Router 7 (lazy-loaded pages) |
| Visualization | react-force-graph-2d (graph/factions), react-leaflet + Leaflet (geographic map) |
| Backend | Python 3.9+ + FastAPI (async) |
| Database | SQLite (aiosqlite) — single file at `~/.ai-reader-v2/data.db` |
| Vector DB | ChromaDB + BAAI/bge-base-zh-v1.5 embeddings |
| LLM | Ollama (local, default qwen3:8b) or OpenAI-compatible API (cloud) |
| Chinese NLP | jieba (entity pre-scan word segmentation) |
| Package mgmt | npm (frontend), uv (backend) |

## Quick Start

```bash
# Backend
cd backend && uv sync && uv run uvicorn src.api.main:app --reload

# Frontend (separate terminal)
cd frontend && npm install && npm run dev
```

- Frontend dev server: http://localhost:5173 (proxies `/api` and `/ws` to backend)
- Backend dev server: http://localhost:8000
- Ollama must be running locally on port 11434 (or configure `LLM_PROVIDER=openai` for cloud mode)

## Project Structure

```
AI-Reader-V2/
├── backend/
│   ├── pyproject.toml              # Python deps (uv)
│   └── src/
│       ├── api/
│       │   ├── main.py             # FastAPI app entry, CORS, lifespan
│       │   ├── routes/             # REST endpoints (15 routers)
│       │   └── websocket/          # WS handlers (analysis progress, chat streaming)
│       ├── services/               # Business logic (14 services)
│       │   └── geo_skills/         # Geographic Agent Skills (Edmonds + snapshots)
│       ├── extraction/             # LLM fact extraction + entity pre-scan pipeline
│       │   ├── entity_pre_scanner.py  # jieba stats + LLM classification
│       │   └── prompts/            # System prompt + few-shot examples
│       ├── db/                     # SQLite + ChromaDB data access
│       ├── models/                 # Pydantic schemas / dataclasses
│       ├── infra/                  # Config, LLM clients, context budget auto-scaling
│       └── utils/                  # Text processing, chapter splitting
├── frontend/
│   ├── package.json
│   ├── vite.config.ts              # Vite + Tailwind + proxy config
│   └── src/
│       ├── app/                    # App entry, router, layout
│       ├── pages/                  # Page components (12 routed)
│       ├── components/
│       │   ├── ui/                 # shadcn/ui base components
│       │   ├── shared/             # Reusable components (UploadDialog, ScenePanel, etc.)
│       │   ├── entity-cards/       # Entity card drawer system
│       │   ├── visualization/      # Graph, map, timeline, geography panel components
│       │   └── chat/               # Chat UI components
│       ├── stores/                 # Zustand stores (7 stores)
│       ├── api/                    # REST client + types
│       ├── hooks/
│       └── lib/
└── scripts/                        # Utility scripts (demo export, data tools)
```

## Key Architecture Concepts

### ChapterFact — Core Data Model

The system's central concept. Each chapter produces one `ChapterFact` JSON containing: characters, relationships, locations, item_events, org_events, events, new_concepts. Stored as JSON text in `chapter_facts.fact_json`. Entity profiles (PersonProfile, LocationProfile, etc.) are **aggregated on-the-fly** from ChapterFacts — not persisted as separate tables.

### Entity Pre-Scan (Optional, Before Analysis)

`EntityPreScanner` — Phase 1: jieba word segmentation + n-gram frequency stats + dialogue attribution regex + suffix pattern matching + naming pattern extraction (regex for "叫作/名叫/绰号" patterns) → candidate list. Phase 2: LLM classifies candidates into entity types with aliases. Output: `entity_dictionary` table. The dictionary is injected into the extraction prompt to improve entity recognition quality.

Includes numeric-prefix name recovery (e.g., "二愣子", "三太子") via POS recovery and candidate merging. Naming-source entries bypass the frequency cutoff to ensure explicitly introduced names are always included.

### Analysis Pipeline


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mouseart2025/AI-Reader-V2](https://github.com/mouseart2025/AI-Reader-V2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
