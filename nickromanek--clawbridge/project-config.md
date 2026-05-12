---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is ClawBridge?

ClawBridge is a desktop and browser automation agent with a web dashboard. It orchestrates three AI-powered engines (browser-use, computer-use, OpenClaw) behind a FastAPI server with a WebSocket-driven chat UI. BYOK (bring your own key) — all LLM API keys stay local.

## Development Model: Monolith-First

**`clawbridge.py`** (~12,900 lines) is the single-file monolith and the **only active development target**. All changes go here first. It contains the full application: FastAPI routes, engine implementations, dashboard HTML/JS, safety system, personality manager, task orchestrator, SQLite persistence, and inline CSS/JS.

`clawbridge/` is a modular package that mirrors the monolith's logic split across files. It exists for reference and testing but lags behind the monolith. The test suite (`tests/`) imports from the package, not the monolith.

`clawbridge_mcp.py` is a standalone MCP server that proxies to the monolith's REST API. It has no shared code with the monolith — it's a thin HTTP client wrapper.

## Commands

```bash
# Run the server (primary way)
python clawbridge.py
# Dashboard at http://localhost:8765

# Run with Docker
docker-compose up

# Install dev dependencies
pip install -r requirements-dev.txt

# Run all tests (unit + integration, against the package)
pytest

# Run a single test file
pytest tests/unit/test_safety.py

# Run a single test by name
pytest tests/unit/test_safety.py -k "test_scan_detects_aws_key"

# Run with coverage
pytest --cov=clawbridge

# Smoke tests (requires running server on :8765)
python tests/smoke_test.py

# Build portable Windows distribution
python build.py

# Build Windows installer (needs Inno Setup ISCC.exe on PATH)
python build.py --inno

# Build macOS app
python build_macos.py --arch arm64
```

## Architecture

```
Request flow:
  Dashboard (inline HTML/JS) ──WebSocket──▶ FastAPI ──▶ TaskManager ──▶ Engine
                                                            │
                                                    ┌───────┼───────┐
                                                    ▼       ▼       ▼
                                              browser-use  computer  openclaw
                                              (Playwright) (pyauto   (Node.js
                                               + LLM)      gui+UIA)  gateway)
```

### Engine selection
- `auto` (default): LLM task planner (`_plan_task()`) decomposes prompts into 1-3 steps with engine assignments. All web tasks route to browser-use, desktop apps to computer-use, chat to openclaw. Falls back to keyword heuristic (`_engine_for()`) if planner times out (3s).
- Each engine implements `EngineBase(ABC)` with `run_task()` and `get_status()`

### Key subsystems in the monolith
- **Settings** — Pydantic model loading from `.env`. BYOK key management.
- **TaskManager** — Async queue with configurable concurrency (`MAX_CONCURRENT_TASKS`). Handles engine routing, personality context injection, safety screening, retry with exponential backoff, task-level timeouts, and automatic pending task promotion.
- **PersonalityManager** — File-based identity system (`workspace/SOUL.md`, `IDENTITY.md`, `USER.md`, `MEMORY.md`, daily logs). Context assembled by `get_system_context()` and injected into each engine differently.
- **Safety/Policy** — `safety_scan_prompt()` detects credentials/PII/injection. `safety_redact()` scrubs before logging. Three policy modes: permissive, guarded (default), strict.
- **AuditLogger** — SQLite with tables: `tasks`, `task_steps`, `audit_log`, `replay_outcomes`, `planner_items`. Step traces persisted for replay. Outcome data powers confidence model learning.
- **Planner** — Kanban-style checklist in the dashboard for tracking project work. SQLite-backed with phases. API: `GET/POST/PUT/DELETE /api/planner`, `POST /api/planner/seed`. Items use zero-padded numeric IDs (01, 02, 03...) for quick `/do` command access. Multiple presets: Developer (24 items), Demo (8), Real Estate (7), QA Testing (7), Content Creation (7). POST `/api/planner` accepts optional `id` field for custom IDs (falls back to UUID). Phase filter dropdown on kanban view filters cards by phase.
- **Failure Analysis** — `analyze_task_failure()` algorithmically diagnoses failed tasks by parsing step traces. Detects: repeated actions (3+ same action at same coordinates), stale sequences, max-steps hit, hard-stops. Auto-populated on task ERROR. Displayed as color-coded timeline in dashboard history view.
- **Dashboard HTML** — ~1,800 lines of inline HTML/JS/CSS rendered by `_dashboard_html()`. Server-side rendering via `window.__PRELOAD__` pattern. Includes slash command autocomplete, always-visible stop button, and chat-integrated workflow save card.
- **WorkflowManager** — Desktop action recording (pynput) and adaptive replay with accessibility-tree element matching + LLM fallback.

### Task Queue & Reliability

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NickRomanek/clawbridge](https://github.com/NickRomanek/clawbridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
