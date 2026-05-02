---
trigger: always_on
description: This file is for AI agents (Claude Code, etc.) working on the Wairz codebase. It describes the architecture, conventions, and patterns you need to follow when making changes.
---

# CLAUDE.md — Wairz Codebase Guide

This file is for AI agents (Claude Code, etc.) working on the Wairz codebase. It describes the architecture, conventions, and patterns you need to follow when making changes.

**What is Wairz?** An open-source, browser-based firmware reverse engineering and security assessment platform. Users upload firmware, the tool unpacks it, and provides a unified interface for filesystem exploration, binary analysis, emulation, fuzzing, and security assessment — augmented by an AI assistant connected via MCP (Model Context Protocol). See `README.md` for user-facing documentation.

---

## Architecture Overview

```
Claude Code / Claude Desktop
        │
        │ MCP (stdio)
        ▼
┌─────────────────┐     ┌──────────────────────────────────┐
│   wairz-mcp     │────▶│         FastAPI Backend           │
│  (MCP server)   │     │                                    │
│  60+ tools      │     │  Services: firmware, file,         │
│                 │     │  analysis, emulation, fuzzing,     │
│  Entry point:   │     │  sbom, uart, finding, export...    │
│  wairz-mcp CLI  │     │                                    │
└─────────────────┘     │  Ghidra headless · QEMU · AFL++    │
                        └──────────┬───────────────────────┘
                                   │
┌──────────────┐    ┌──────────────┼──────────────┐
│   React SPA  │───▶│  PostgreSQL  │  Redis       │
│  (Frontend)  │    │              │              │
└──────────────┘    └──────────────┴──────────────┘

Host machine (optional):
  wairz-uart-bridge.py ←─ TCP:9999 ─→ Docker backend
```

- **Frontend:** React 19 + Vite + TypeScript, shadcn/ui + Tailwind, Monaco Editor, ReactFlow, xterm.js, Zustand
- **Backend:** Python 3.12 + FastAPI (async), SQLAlchemy 2.0 (async) + Alembic, pydantic-settings
- **MCP Server:** `wairz-mcp` CLI entry point (`app.mcp_server:main`), stdio transport, 60+ tools
- **Database:** PostgreSQL 16 (JSONB for analysis cache)
- **Containers:** Docker Compose — backend, postgres, redis, emulation (QEMU), fuzzing (AFL++)

---

## Directory Structure

```
wairz/
├── backend/
│   ├── pyproject.toml           # Entry point: wairz-mcp
│   ├── alembic/versions/        # Database migrations (auto-run on container start)
│   └── app/
│       ├── main.py              # FastAPI app + router registration
│       ├── config.py            # Settings via pydantic-settings
│       ├── database.py          # Async engine, session factory, get_db dependency
│       ├── mcp_server.py        # MCP server with dynamic project switching
│       ├── models/              # SQLAlchemy ORM models
│       ├── schemas/             # Pydantic request/response schemas
│       ├── routers/             # FastAPI REST endpoint routers
│       ├── services/            # Business logic layer
│       ├── workers/             # Background tasks (firmware unpacking)
│       ├── ai/
│       │   ├── __init__.py      # Tool registry factory — registers all tool categories
│       │   ├── tool_registry.py # ToolContext + ToolRegistry framework
│       │   ├── system_prompt.py # MCP system prompt for Claude
│       │   └── tools/           # Tool handlers by category
│       └── utils/
│           ├── sandbox.py       # Path traversal prevention (CRITICAL)
│           └── truncation.py    # Output truncation (30KB max)
├── frontend/
│   └── src/
│       ├── pages/               # Route pages, registered in App.tsx
│       ├── components/          # UI components organized by feature
│       ├── api/                 # Axios API client functions
│       ├── stores/              # Zustand state management
│       └── types/               # TypeScript type definitions
├── ghidra/
│   ├── Dockerfile
│   └── scripts/                 # Custom Java analysis scripts for headless Ghidra
├── emulation/
│   ├── Dockerfile               # QEMU + kernels (ARM, MIPS, MIPSel, AArch64)
│   └── scripts/                 # start-user-mode.sh, start-system-mode.sh, serial-exec.sh
├── fuzzing/
│   └── Dockerfile               # AFL++ with QEMU mode
└── scripts/
    └── wairz-uart-bridge.py     # Host-side serial bridge (standalone, pyserial only)
```

---

## How to Add Things

### Adding a New MCP Tool

1. Create or edit a handler in `backend/app/ai/tools/<category>.py`:
   ```python
   async def _handle_my_tool(input: dict, context: ToolContext) -> str:
       # Available on context: project_id, firmware_id, extracted_path, db
       path = context.resolve_path(input.get("path", "/"))  # validates against sandbox
       # ... do work ...
       return "result string (max 30KB, truncated automatically)"
   ```
2. Register in the same file's `register_<category>_tools(registry)` function:
   ```python
   registry.register(name="my_tool", description="...", input_schema={...}, handler=_handle_my_tool)
   ```
3. If it's a new category file, import and call `register_<category>_tools(registry)` in `backend/app/ai/__init__.py`.

### Adding a New REST Endpoint

1. Create router: `backend/app/routers/<name>.py`
   ```python
   router = APIRouter(prefix="/api/v1/projects/{project_id}/<name>", tags=["<name>"])
   ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [digitalandrew/wairz](https://github.com/digitalandrew/wairz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
