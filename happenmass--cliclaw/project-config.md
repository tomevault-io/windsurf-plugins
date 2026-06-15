---
trigger: always_on
description: Provides execution control for the MainAgent loop:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Cliclaw

Cliclaw is a chat-based meta-orchestrator that commands coding agents (like Claude Code) via tmux. It runs as a persistent HTTP + WebSocket server with a web chat UI. The MainAgent can hold natural conversations and autonomously execute complex development tasks by commanding coding agents in tmux sessions.

Core flow: **Chat message → MainAgent (IDLE ↔ EXECUTING state machine) → Streaming LLM → Tool execution in tmux → Response via WebSocket**

## Commands

```bash
npm run build          # tsc — compile to dist/
npm run dev            # tsc --watch
npm test               # vitest run — all tests
npm run test:watch     # vitest — watch mode
npx vitest test/core/main-agent.test.ts   # run a single test file
npm run check          # biome check src/
npm run format         # biome format --write src/
npm start              # node dist/main.js — starts the server on port 3120
```

## Code Style

- **Formatter**: Biome — tabs, indent width 3, line width 120
- **Module system**: ESM (`"type": "module"` in package.json)
- **TypeScript**: strict mode, target ES2022, module Node16
- **Imports**: use `.js` extension in relative imports (Node16 module resolution)
- `noExplicitAny: off`, `noNonNullAssertion: off` — these are intentionally relaxed
- Use `useConst: error` — always prefer `const`

## Architecture

### Entry Point (`src/main.ts`) and CLI (`src/cli.ts`)
`cli.ts` exports `parseCliArgs()` for CLI argument parsing (--agent, --provider, --model, --base-url, --port, --cwd, etc.) and `printHelp()`/`printVersion()`. `main.ts` orchestrates startup:
1. **Bootstrap** — MemoryStore (SQLite), EmbeddingProvider (auto-fallback), initial memory file sync, skill discovery → filter → registry, ConversationStore initialization, CommandRegistry setup
2. **Restore** — If SQLite has existing messages, restore conversation into ContextManager
3. **Serve** — Start Express + WebSocket server on configurable port (default 3120)
4. **Shutdown** — SIGINT/SIGTERM triggers graceful shutdown (stop agent → close server → close DB)

Subcommands: `config`, `doctor`, `init`, `remember` are handled before server startup.

### MainAgent (`src/core/main-agent.ts`)
Chat-driven decision engine with a two-state machine: **IDLE** ↔ **EXECUTING**.

- **IDLE**: Waits for user messages via `handleMessage(content)`. Streams LLM response. If LLM returns tool calls → transitions to EXECUTING. If pure text → stays IDLE.
- **EXECUTING**: Self-loop executing tool calls. Between rounds: checks `stopRequested`, drains `MessageQueue` (human messages queued during execution), checks context thresholds. Terminal tools (`mark_failed`, `escalate_to_human`) return to IDLE. When the LLM responds with text only (no tool calls), it naturally returns to IDLE.

Uses `llmClient.stream()` for all LLM calls — text deltas are broadcast to WebSocket clients in real-time.

Emits events: `state_change`, `log`. Built-in tools:
- `send_to_agent` / `respond_to_agent` — interact with coding agent in tmux (both have required `summary` parameter for chat UI updates)
- `inspect_agent` — capture agent pane content and task status
- `list_agent_tasks` — list active sub-agent tasks and pending events
- `mark_failed` — terminal: return to IDLE
- `escalate_to_human` — terminal: request human intervention
- `memory_search` / `memory_get` — hybrid search and read memories
- `memory_edit` — edit memory files (modes: append, overwrite, replace, delete). `memory_write` is a backwards-compatible alias
- `persistent_memory` — read/update global and project MEMORY.md (sections: user_profile, project_conventions, key_decisions, people_and_context, active_notes)
- `read_skill` — read full SKILL.md content on demand
- `create_agent` — create a `cliclaw-` prefixed tmux session and launch agent
- `list_agents` — list all `cliclaw-` prefixed agents
- `kill_agent` — gracefully exit agent, destroy tmux session, and clean up registry; returns resume id; supports "all"
- `exec_command` — execute read-only bash commands for reconnaissance

### Server Layer (`src/server/`)
HTTP + WebSocket server for the chat interface.

- `index.ts` — Express app creation, static file serving (`web/`), REST API (`/api/history`, `/api/status`), WebSocket server on `/ws` path. `startServer()` returns a `ServerInstance` with a `close()` method.
- `chat-broadcaster.ts` — Manages WebSocket client connections. `broadcast(message)` sends to all connected clients. Used by MainAgent to push `assistant_delta`, `assistant_done`, `agent_update`, `tool_activity`, `state`, `system`, `clear` messages.
- `ws-handler.ts` — Handles individual WebSocket connections. Routes `{ type: "message" }` to `MainAgent.handleMessage()` and `{ type: "command" }` to `CommandRouter`. Sends current state on connect.
- `command-router.ts` — Handles slash commands (`/stop`, `/resume`, `/clear`, `/reset`, `/compact`, `/context`, `/tidy`). `/tidy` uses LLM to review memory files and archive outdated entries. Receives optional `llmClient`, `promptLoader`, `memoryStore`, `syncMemory` dependencies for commands that need LLM access.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Happenmass/Cliclaw](https://github.com/Happenmass/Cliclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
