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
│   └── integrations/    # Typed native integrations (GitHub, future: Jira/Linear)
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

| Task                  | Location                                                                                                                        | Notes                                                                                                                      |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Add CLI command       | `src/kagan/cli/`                                                                                                                | Click group in `main.py`, lazy-loaded modules                                                                              |
| Add MCP tool          | `src/kagan/mcp/toolsets/`                                                                                                       | One file per domain, use `get_context()`                                                                                   |
| Add TUI screen        | `src/kagan/tui/screens/`                                                                                                        | Register in `app.py` SCREENS dict                                                                                          |
| Add TUI widget        | `src/kagan/tui/widgets/`                                                                                                        | Follow Textual compose pattern                                                                                             |
| Modify task lifecycle | `src/kagan/core/_transitions.py`                                                                                                | State machine for task status                                                                                              |
| Add agent backend     | `src/kagan/core/_agent.py`                                                                                                      | AGENT_BACKENDS registry dict                                                                                               |
| Add DB migration      | `alembic -c alembic.ini revision --autogenerate -m "msg"`                                                                       | Via `poe db-migration-generate`                                                                                            |
| Wire protocol change  | `src/kagan/server/responses.py`                                                                                                 | Response models → JSON Schema → TypeScript via `scripts/generate_wire_types.py`                                            |
| Add SSE event channel | `src/kagan/server/_event_routes.py`                                                                                             | Register route + hook into `register_event_routes(mcp)` in `server.py`                                                     |
| Add frame consumer    | `src/kagan/tui/_event_source.py` / `packages/web/src/lib/hooks/use-entry-stream.ts` / `packages/vscode/src/api/event-source.ts` | TUI: implement `InProcEventSource`/`HttpEventSource` interface; Web: use `useEntryStream`; VS Code: use `KaganEventSource` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kagan-sh/kagan-legacy](https://github.com/kagan-sh/kagan-legacy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
