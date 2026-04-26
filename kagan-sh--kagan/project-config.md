---
trigger: always_on
description: **Generated:** 2026-03-13
---

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-03-13
**Commit:** ddf5c5f
**Branch:** feat/remote-clients

## OVERVIEW

Kagan — AI-powered Kanban TUI (Python 3.12+/Textual) that orchestrates coding agents on your codebase. Supports 14 agent backends, auto/pair execution modes, MCP protocol, and a bundled web dashboard (React 19).

## STRUCTURE

```
kagan/
├── src/kagan/           # Python package (core logic, TUI, CLI, MCP, server)
│   ├── core/            # Domain: DB, models, agents, tasks, sessions, worktrees
│   ├── tui/             # Textual TUI: screens/, widgets/, styles/
│   ├── cli/             # Click CLI surface (entrypoint: `kagan`/`kg`)
│   ├── mcp/             # MCP server: toolsets/, prompts, resources
│   ├── server/          # HTTP server: REST API, SSE streaming, auth, web UI
│   ├── chat/            # CLI chat REPL: ACP streaming, commands, sessions
│   ├── crypto/          # X25519 key exchange, TLS, tokens, QR
│   ├── wire/            # (compat shim) Re-exports envelope types
│   ├── integrations/    # GitHub integration
│   └── plugins/         # Plugin system (entry-point based)
├── packages/
│   ├── vscode/          # VS Code extension: chat participant, tree view, SCM, reviews
│   ├── web/             # React 19 + jotai + Tailwind CSS 4 web dashboard (SPA)
│   └── wire/            # (removed — TS types now generated from response models)
├── tests/               # pytest: core/, tui/, mcp/, server/, unit/, helpers/
├── scripts/             # Build/quality scripts (LOC budget, web build)
├── docs/                # MkDocs documentation site
├── registry/            # Persona repo whitelist
└── references/          # External reference repos (NOT part of build)
```

## WHERE TO LOOK

| Task                  | Location                                                  | Notes                                                                             |
| --------------------- | --------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Add CLI command       | `src/kagan/cli/`                                          | Click group in `main.py`, lazy-loaded modules                                     |
| Add MCP tool          | `src/kagan/mcp/toolsets/`                                 | One file per domain, use `get_context()`                                          |
| Add TUI screen        | `src/kagan/tui/screens/`                                  | Register in `app.py` SCREENS dict                                                 |
| Add TUI widget        | `src/kagan/tui/widgets/`                                  | Follow Textual compose pattern                                                    |
| Modify task lifecycle | `src/kagan/core/_transitions.py`                          | State machine for task status                                                     |
| Add agent backend     | `src/kagan/core/_agent.py`                                | AGENT_BACKENDS registry dict                                                      |
| Add DB migration      | `alembic -c alembic.ini revision --autogenerate -m "msg"` | Via `poe db-migration-generate`                                                   |
| Wire protocol change  | `src/kagan/server/responses.py`                           | Response models → JSON Schema → TypeScript via `scripts/generate_wire_types.py`   |
| Web UI feature        | `packages/web/src/`                                       | React 19 + jotai + Tailwind CSS 4                                                 |
| API endpoint          | `src/kagan/server/_routes.py`                             | Starlette routes via FastMCP                                                      |
| VS Code feature       | `packages/vscode/src/providers/`                          | One provider per VS Code API surface                                              |
| VS Code command       | `packages/vscode/src/commands/`                           | Register in `extension.ts`                                                        |
| Modify prompt system  | `src/kagan/core/_prompts.py`                              | Three-layer resolution: dotfile → defaults + behavioral → additional instructions |

## CONVENTIONS

- **Private modules**: Underscore prefix (`_tasks.py`, `_agent.py`) = internal; `client.py`, `models.py`, `enums.py` = public API
- **DB access**: Always via `_db_sync`/`_db_async` helpers wrapping SQLModel sessions
- **Async pattern**: `asyncio.to_thread` for DB ops, native async for I/O
- **Logging**: `loguru.logger` everywhere (not stdlib logging)
- **Error hierarchy**: All errors inherit `KaganError` (see `core/errors.py`)
- **LOC budget**: 2500 lines max per Python file (enforced via `poe check-loc`)
- **Complexity cap**: McCabe max-complexity = 20 (ruff C90)
- **Type annotations**: All public functions typed; pyrefly for typechecking (not mypy)
- **MCP annotations**: TC001/TC002/TC003 suppressed in `src/kagan/mcp/` — MCP evaluates annotations at runtime
- **Prompt resolution**: Three-layer pipeline in `core/_prompts.py` — dotfile override → code defaults + behavioral settings → additional instructions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kagan-sh/kagan](https://github.com/kagan-sh/kagan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
