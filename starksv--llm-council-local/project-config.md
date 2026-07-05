---
trigger: always_on
description: Technical notes, architecture decisions, and implementation details for development sessions.
---

# CLAUDE.md — LLM Council (Local Ollama Edition)

Technical notes, architecture decisions, and implementation details for development sessions.

## Project Overview

LLM Council is a 3-stage deliberation system where multiple local Ollama models collaboratively answer user questions. Key innovation: anonymized peer review in Stage 2 prevents models from playing favourites.

## Architecture

### Backend (`backend/`)

**`config.py`**
- `COUNCIL_MODELS` — list of Ollama model tags (e.g. `"deepseek-r1:8b"`)
- `CHAIRMAN_MODEL` — model that synthesises the final answer
- `OPENROUTER_API_URL` — Ollama endpoint: `http://localhost:11434/v1/chat/completions`
- No API key required for local Ollama; `OPENROUTER_API_KEY` from `.env` is optional and only sent if present
- Backend runs on **port 8001**

**`ollama.py`** *(was `openrouter.py` — renamed via `git mv`)*
- `query_model(model, messages, timeout=360.0)` — single async model query
- Always returns a dict: `{'content': ..., 'reasoning_details': ...}` on success OR `{'error': True, 'error_type': 'timeout'|'api_error'|'unknown', 'error_msg': str}` on failure. Never returns `None`.
- Only adds `Authorization` header when `OPENROUTER_API_KEY` is set (Ollama ignores it, but avoids `Bearer None`)
- `query_models_sequential(models, messages)` — queries models one at a time. **Sequential, not parallel** — required for single-GPU Ollama to avoid VRAM exhaustion and enables per-model SSE progress events.

**`council.py`** — Core orchestration
- `_extract_thinking(text)` — strips DeepSeek `<think>...</think>` blocks, returns `(clean_response, thinking_or_None)`
- `run_council_streaming()` — async generator emitting fine-grained SSE events:
  - `stage1_start`, `stage1_model_querying`, `stage1_model_done`, `stage1_model_error`
  - `stage2_start`, `stage2_model_querying`, `stage2_model_done`, `stage2_model_error`
  - `stage3_querying`, `stage3_complete` (includes `elapsed_seconds`)
- `parse_ranking_from_text()` — extracts "FINAL RANKING:" section; fallback regex handles non-standard formats
- `calculate_aggregate_rankings()` — computes average rank position for the Street Cred leaderboard
- `generate_conversation_title()` — auto-titles new conversations from the first user message

**`storage.py`**
- JSON files in `data/conversations/`
- `update_conversation_title(id, title)` — used for both auto-titling and manual renames
- `delete_conversation(id)` — removes the JSON file
- All metadata (`label_to_model`, `aggregate_rankings`, `elapsed_seconds`) **is persisted** to storage so historical conversations can show the leaderboard and elapsed time

**`main.py`** — FastAPI app
- CORS for `localhost:5173` and `localhost:3000`
- `POST /api/conversations` — create
- `GET /api/conversations` — list
- `GET /api/conversations/{id}` — load full conversation
- `PATCH /api/conversations/{id}/title` — rename conversation (body: `{"title": "..."}`)
- `DELETE /api/conversations/{id}` — delete
- `POST /api/conversations/{id}/message` — non-streaming council run
- `POST /api/conversations/{id}/message/stream` — **streaming SSE** council run (primary path)

### Frontend (`frontend/src/`)

**Stack:** React + Vite + **Tailwind v3** (utility classes, `preflight: false`) + custom CSS for complex stage styling

**`constants.js`** — `MODEL_DISPLAY_NAMES` map from raw Ollama tags to readable names; `displayName(model)` helper used everywhere.

**`api.js`** — All fetch calls: `listConversations`, `createConversation`, `getConversation`, `renameConversation` (PATCH), `deleteConversation`, `sendMessage`, `sendMessageStream` (SSE reader)

**`App.jsx`**
- Top-level state: conversations list, current conversation, loading, `sidebarOpen`
- `updateLastAssistant(prev, updater)` helper for immutable state updates during SSE streaming
- `handleRenameConversation(id, newTitle)` — calls `api.renameConversation`, updates both lists
- Mobile overlay (`fixed inset-0 z-40 bg-black/50 md:hidden`) when sidebar is open on small screens

**`components/Sidebar.jsx`**
- Collapsible: `isOpen` prop + `onToggle`. Collapses to 44px icon rail on desktop; overlays content on mobile.
- Inline rename: double-click title or click ✎ button → input appears. Enter/blur to save, Escape to cancel.
- Action buttons (rename ✎, delete ✕) fade in on item hover via `.conv-actions`.

**`components/ChatInterface.jsx`**
- `StageProgress` inline component: per-model status with pulsing dots during loading
- Mobile topbar with ☰ hamburger (hidden on `md:` and up via Tailwind)
- Always-visible input bar at bottom; Enter sends, Shift+Enter newline

**`components/Stage1.jsx`** — Blue theme; collapsible header; pill tabs; "Show/Hide reasoning" toggle for DeepSeek thinking; failed model pills with error type. `remarkPlugins={[remarkGfm]}` on all ReactMarkdown.

**`components/Stage2.jsx`** — Amber theme; collapsible; Street Cred leaderboard shown first; de-anonymised evaluation tabs; parsed ranking block below raw text.

**`components/Stage3.jsx`** — Green theme; collapsible; "Council deliberated for Xs" elapsed badge; chairman model pill.

## Key Design Decisions

### Sequential queries

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [starkSV/llm-council-local](https://github.com/starkSV/llm-council-local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
