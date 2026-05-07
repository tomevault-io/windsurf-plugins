---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

ChatCrystal is an AI conversation knowledge crystallization tool. It imports conversations from AI coding tools (Claude Code, Cursor, Codex CLI, Trae, GitHub Copilot), uses LLM to generate structured notes (title, summary, key conclusions, code snippets, tags), and provides semantic search via embeddings. The UI is in Simplified Chinese.

## Commands

```bash
# Development (server port 3721 + client port 13721)
npm run dev

# Build for production
npm run build

# Production server (serves frontend statically on port 3721)
npm start

# Electron desktop app
npm run dev:electron        # dev mode (Vite HMR + Electron window)
npm run build:electron      # build NSIS installer → release/
npm run pack:electron       # build unpacked directory (faster for testing)

# Release (bump version + git tag + push → CI builds & publishes)
npm run release             # patch bump (0.1.0 → 0.1.1)
npm run release -- minor    # minor bump
npm run release -- major    # major bump
npm run release -- 1.0.0    # explicit version

# Lint
npm run lint
npm run lint:fix
```

### CLI (`crystal`)

Install globally: `npm install -g chatcrystal`

```bash
# Core commands
crystal status                          # Server status and DB stats
crystal import [--source claude-code]   # Scan and import conversations
crystal search "query" [--limit 10]     # Semantic search
crystal notes list [--tag X]            # Browse notes
crystal notes get <id>                  # View note detail
crystal notes relations <id>            # View note relations
crystal tags                            # List tags with counts
crystal summarize <id>                  # Summarize one conversation
crystal summarize --all                 # Batch summarize
crystal config get                      # View config
crystal config set llm.provider openai  # Update config
crystal config test                     # Test LLM connection

# Server management
crystal serve                           # Start server (foreground)
crystal serve -d                        # Start server (daemon)
crystal serve stop                      # Stop daemon
crystal serve status                    # Check if running

# MCP Server (for AI tool integration)
crystal mcp                             # Start MCP stdio server
```

Global options: `--base-url` (server URL), `--json` (force JSON output), `--version`.

Auto-start: commands that need the server will auto-launch it in background if not running.

#### MCP Configuration

Codex (`settings.json`):
```json
{
  "mcpServers": {
    "chatcrystal": {
      "command": "crystal",
      "args": ["mcp"]
    }
  }
}
```

MCP exposes 6 tools: read-only knowledge tools `search_knowledge`, `get_note`, `list_notes`, `get_relations`, plus memory-loop tools `recall_for_task` and `write_task_memory`.

## Architecture

Monorepo with three npm workspaces:

### `shared/` — Shared Types (`@chatcrystal/shared`)
- No build step; exports TypeScript types directly from `types/index.ts`

### `server/` — Fastify Backend (`@chatcrystal/server`)
- **Runtime:** tsx (dev + prod)
- **Framework:** Fastify v5 with CORS and static file serving (production SPA fallback)
- **Database:** sql.js (WASM SQLite), stored under the active runtime data directory as `chatcrystal.db`, auto-saved every 30s
- **Key modules:**
  - `db/` — Schema (7 tables), utils (`resultToObjects`)
  - `parser/` — Plugin architecture via `SourceAdapter`. Five built-in adapters:
    - `adapters/claude-code.ts` — JSONL from `~/.claude/projects/`. Sanitizes `<system-reminder>`, `<command-name>` tags.
    - `adapters/codex.ts` — JSONL event stream from `~/.codex/sessions/`. Reconstructs conversation from event_msg/response_item events.
    - `adapters/cursor.ts` — SQLite `state.vscdb` from Cursor's workspaceStorage/globalStorage. Reads composer metadata + bubble data via sql.js.
    - `adapters/trae.ts` — SQLite `state.vscdb` from Trae's workspaceStorage. Reads `memento/icube-ai-agent-storage` key; extracts content from agentTaskContent for agent responses.
    - `adapters/copilot.ts` — JSONL from VS Code's workspaceStorage/chatSessions + globalStorage/emptyWindowChatSessions. Parses session snapshots (kind:0) with requests/response arrays.
  - `services/llm.ts` — Provider factory via Vercel AI SDK (Ollama, OpenAI, Anthropic, Google, Azure, Custom)
  - `services/summarize.ts` — Conversation preprocessing (truncate ~8000 tokens) + LLM call + JSON parsing + DB persistence. Auto-generates embeddings after summarization.
  - `services/embedding.ts` — Embedding model factory + vectra LocalIndex + text chunking
  - `services/import.ts` — Scan + dedup (file size + mtime) + batch insert
  - `routes/` — REST endpoints: status, config, import, conversations CRUD, notes CRUD, tags, search, queue status, batch operations
  - `queue/` — p-queue singleton (concurrency=1, 1 req/sec, 429 retry)
  - `watcher/` — chokidar watches all configured data source paths (Claude Code, Codex CLI, Cursor, Trae, GitHub Copilot), debounced auto-import

### `client/` — React SPA
- **Build:** Vite v8 + `@vitejs/plugin-react`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZengLiangYi/ChatCrystal](https://github.com/ZengLiangYi/ChatCrystal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
