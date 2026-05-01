---
trigger: always_on
description: This file provides guidance to AI coding agents working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working with code in this repository.

## What is Helmor

Helmor is a local-first desktop app built with **Tauri v2** (Rust backend) + **React 19** + **Vite** + **TypeScript**. It provides a workspace management UI with its own SQLite database (`~/helmor/` in release, `~/helmor-dev/` in debug), letting users browse workspaces/sessions/messages and send prompts to AI agents (Claude Code CLI, OpenAI Codex CLI) via streaming IPC.

## Commands

```bash
bun install                  # Install deps (bun 1.3+). Also runs `bun install` in sidecar/ via postinstall.
bun run dev                  # Full desktop app: Tauri + Vite (localhost:1420 in webview)
bun run dev:analyze          # Same as dev, with perf HUD (VITE_HELMOR_PERF_HUD=1)
bun run build                # tsc + vite build (frontend bundle to dist/)
bun run typecheck            # tsc --noEmit for frontend AND sidecar
bun run lint                 # biome check . + cargo clippy -- -D warnings
bun run lint:fix             # biome --write + cargo clippy --fix + cargo fmt
```

Tests are **three targets** — `bun run test` runs all three (frontend -> sidecar -> rust). Pre-commit hook runs biome on JS/TS and clippy/fmt on Rust.

```bash
bun run test                 # All three suites
bun run test:frontend        # vitest run (jsdom, @testing-library/react)
bun run test:sidecar         # cd sidecar && bun test
bun run test:rust            # cd src-tauri && cargo test
bun run test:rust:update-snapshots    # INSTA_UPDATE=always
bun run test:watch           # vitest watch (frontend only)
```

Single test file: `bun x vitest run src/App.test.tsx` | `cd sidecar && bun test src/foo.test.ts` | `cd src-tauri && cargo test --test pipeline_scenarios -- <name>`

## Architecture

### Three-process model

- **Frontend** (`src/`): React 19 SPA in Tauri webview. Root state in `App.tsx` via `useState` + TanStack React Query + context providers.
- **Rust backend** (`src-tauri/src/`): Tauri host, SQLite database, spawns and supervises the sidecar.
- **Sidecar** (`sidecar/`): Bun + TypeScript, wraps `@anthropic-ai/claude-agent-sdk` and `@openai/codex-sdk`. Built to `sidecar/dist/helmor-sidecar` via `bun build --compile`. JSON event stream over stdout.

Message flow: user prompt -> Rust `agents::streaming` -> sidecar -> SDK -> stdout events -> Rust accumulator -> adapter + collapse -> `ThreadMessageLike[]` -> `tauri::ipc::Channel` -> React.

### Frontend structure (`src/`)

Feature-based layout. Each feature folder follows: `index.tsx` (main) + `container.tsx` (data/state) + `hooks/` + tests.

| Path | Role |
| --- | --- |
| `App.tsx` | Root. Owns selection state, view mode, sending status. |
| `features/panel/` | Chat thread container, header, message components, thread viewport. |
| `features/conversation/` | Conversation renderer + `use-streaming` hook. |
| `features/composer/` | Lexical-based message input. Plugins in `editor/plugins/`. |
| `features/editor/` | Monaco file editor surface. |
| `features/inspector/` | Right-side inspector (actions, changes sections). |
| `features/navigation/` | Sidebar workspace groups. |
| `features/commit/` | Commit button + lifecycle hook. |
| `features/settings/` | Settings dialog + panels (CLI install, repo settings, Conductor import). |
| `shell/` | Top-level layout, GitHub identity gate, panel resize hooks. |
| `components/ai/` | AI-specific components (code block, file tree, reasoning). |
| `components/ui/` | shadcn/ui primitives (base-nova). |
| `lib/api.ts` | IPC bridge -- every Tauri `invoke()` call wrapped as a typed function. |
| `lib/query-client.ts` | React Query keys + query options factories. |
| `lib/settings.ts` | App settings context with Tauri storage. |

### Backend structure (`src-tauri/src/`)

| Module | Role |
| --- | --- |
| `lib.rs` | Tauri app builder. Registers commands, runs setup hook. |
| `commands/` | Tauri command handlers split by domain (session, repository, workspace, editor, github, conductor, settings, system). |
| `agents/` | Agent streaming + persistence (catalog, persistence, queries, streaming, support). |
| `pipeline/` | Message pipeline: `accumulator/` -> `adapter/` + `collapse` -> `ThreadMessageLike[]`. Includes `event_filter.rs`, `classify.rs`, `types.rs`. |
| `workspace/` | Workspace operations (branching, lifecycle, helpers) + `files/` sub-module (editor, changes, types). |
| `git/` | Git operations (ops, watcher). |
| `github/` | GitHub integration (auth, CLI, GraphQL). |
| `models/` | Persistence layer (db, repos, sessions, settings, workspaces). |
| `service.rs` | Service layer. |
| `sidecar.rs` | Sidecar process manager (spawn, stdio, graceful SIGTERM). |
| `schema.rs` | DB schema + idempotent migrations. |
| `mcp.rs` | MCP bridge integration. |
| `logging.rs` | Structured logging setup. |
| `data_dir.rs` | Data dir resolution. `HELMOR_DATA_DIR` env override. |
| `error.rs` | `CommandError` -- bridges `anyhow::Error` to Tauri IPC. |

### Sidecar structure (`sidecar/src/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dohooo/helmor](https://github.com/dohooo/helmor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
