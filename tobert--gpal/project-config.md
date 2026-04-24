---
trigger: always_on
description: Internal documentation for gpal development.
---

# Development Guide

Internal documentation for gpal development.

## Dogfooding

Before committing changes to gpal, use gpal to review them:

```
consult_gemini(
    query="Review server.py for bugs, edge cases, and API misuse",
    model="pro",
    file_paths=["src/gpal/server.py"]
)
```

Gemini catches real issues — see git history for proof.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│              MCP Client                             │
│    (Claude Desktop, Cursor, VS Code, etc.)          │
└─────────────────────┬───────────────────────────────┘
                      │ MCP Protocol
                      ▼
┌─────────────────────────────────────────────────────┐
│                 gpal Server                         │
│                                                     │
│  ┌─────────────────┐  ┌─────────────────┐          │
│  │ consult_gemini  │  │ consult_oneshot │  ← Tools │
│  └────────┬────────┘  └────────┬────────┘          │
│           │                    │                    │
│           └──────────┬─────────┘                    │
│                      ▼                              │
│  ┌──────────────────────────────────────┐          │
│  │       Session Manager                 │          │
│  │  (history preservation, model switch) │          │
│  └──────────────────┬───────────────────┘          │
│                      │                              │
└──────────────────────┼──────────────────────────────┘
                       ▼
┌─────────────────────────────────────────────────────┐
│              Google Gemini API                      │
│                                                     │
│  Gemini has internal tools:                         │
│  • list_directory  • read_file  • search_project   │
│  • url_context    • file_search (when stores exist) │
│                                                     │
│  Automatic Function Calling enabled                 │
│  (Gemini autonomously explores the codebase)        │
└─────────────────────────────────────────────────────┘
```

## Project Structure

```
gpal/
├── src/gpal/
│   ├── __init__.py       # Package metadata (__version__)
│   └── server.py         # MCP server + all logic
├── tests/
│   ├── test_tools.py     # Unit tests (pytest)
│   ├── test_server.py    # Integration tests (FastMCP Client, in-process)
│   ├── test_agentic.py   # Manual: autonomous exploration
│   ├── test_connectivity.py  # Manual: API ping
│   └── test_switching.py     # Manual: Flash→Pro history
└── pyproject.toml        # Dependencies & entry point
```

## Key Design Decisions

### Stateful Sessions

Sessions live in memory (`sessions` dict). Same `session_id` = same conversation. History migrates when switching models.

⚠️ **Limitation**: Sessions are not persisted. Server restart = fresh state.

### Model Strategy

We always prefer the latest and most capable models available from Google.

| Model | Alias | Best For |
|-------|-------|----------|
| `gemini-flash-lite-latest` | `lite` | Cheap exploration (auto mode Phase 1, auto-updates) |
| `gemini-3-flash-preview` | `flash` | Frontier synthesis, searching, analysis |
| `gemini-3.1-pro-preview` | `pro` | Deep reasoning, synthesis, code review |
| `gemini-flash-latest` | — | Web search, code execution (auto-updates) |
| `imagen-4.0-ultra-generate-001` | `imagen` | Ultra quality image generation (default) |
| `imagen-4.0-fast-generate-001` | `imagen-fast` | Fast image generation |
| `gemini-3-pro-image-preview` | `nano-pro` | Best quality images, text rendering, 4K |
| `gemini-2.5-flash-image` | `nano-flash` | Fast, efficient image generation |
| `gemini-2.5-pro-preview-tts` | `speech` | Text-to-speech synthesis (Pro quality) |
| `gemini-2.5-flash-preview-tts` | `speech-fast` | Fast, cheaper text-to-speech |

> **Note**: There is no separate "deep think" model. Gemini thinking mode is enabled via
> `ThinkingConfig(thinking_level="HIGH")` on Pro.

### FastMCP Version

Using FastMCP 3.0.0rc1+. Upgraded from 2.14.x which had a regression in 2.14.5 breaking
async tool functions. The 3.x upgrade was clean — only required dropping `task=True` from
`rebuild_index` (background tasks now need `fastmcp[tasks]` extra).

`ctx.debug/info/warning/error/report_progress` are **async** — must be `await`ed.
`ctx.set_state/get_state` are **sync**.

### FastMCP 3.0 Feature Adoption (v0.3.1)

**Tool Timeouts** — `timeout=` on `@mcp.tool()` decorators. FastMCP uses `anyio.fail_after()`
and converts `TimeoutError` to `McpError(-32000)`.

| Tool | Timeout | Rationale |
|------|---------|-----------|
| `consult_gemini` | 660s | Unified tool (auto mode: Lite explore + synthesis) |
| `consult_gemini_oneshot` | 600s | Stateless queries (Pro+thinking can be slow) |
| `rebuild_index` | 300s | Large index rebuilds |
| All others | None | Quick sync operations |

**Rich ToolResult** — `_consult` returns `ToolResult` (from `fastmcp.tools.tool`) on success
instead of plain `str`. Provides `structured_content` (model ID) and `meta` (model, session_id,
duration_ms) for client introspection. Error paths still return plain strings.

**OTel Simplification** — Removed `opentelemetry-instrumentation-fastapi` (unused — no FastAPI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobert/gpal](https://github.com/tobert/gpal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
