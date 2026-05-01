---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MetaBot — A bridge service that connects IM bots (Feishu/Lark) to the Claude Code Agent SDK. Users chat with Claude Code from Feishu (including mobile), with real-time streaming updates via interactive cards. Runs Claude in `bypassPermissions` mode since there's no terminal for interactive approval.

## Commands

```bash
npm run dev          # Development with tsx (hot reload)
npm run build        # TypeScript compile + build web frontend to dist/
npm run build:web    # Build web frontend only (Vite → dist/web/)
npm start            # Run compiled output (dist/index.js)
```

```bash
npm test             # Run tests (vitest)
npm run lint         # ESLint check
npm run format       # Prettier format
```

## Architecture

The app is a TypeScript ESM project (`"type": "module"`, all imports use `.js` extensions). It connects to Feishu via WebSocket (long connection, no public IP needed) and calls Claude via the `@anthropic-ai/claude-agent-sdk`.

### Message Flow

```
Feishu WSClient → EventHandler (parse, @mention filter) → MessageBridge → ClaudeExecutor → StreamProcessor → Feishu card updates
Web Browser → WebSocket (/ws) → ws-server.ts → MessageBridge.executeApiTask(onUpdate) → WebSocket push → React UI
```

### Key Modules

- **`src/index.ts`** — Entrypoint. Creates Feishu WS client, fetches bot info for @mention detection, wires up the event dispatcher and bridge, handles graceful shutdown.
- **`src/config.ts`** — Loads config. `BotConfig` is the per-bot type; `AppConfig` wraps `{ bots, log }`. `loadAppConfig()` reads `BOTS_CONFIG` JSON file or falls back to single-bot mode from env vars.
- **`src/feishu/event-handler.ts`** — Registers `im.message.receive_v1` on the Lark `EventDispatcher`. Handles text/image parsing, @mention stripping, group chat filtering (only responds when @mentioned, except in 2-member groups which are treated like DMs). Exports `IncomingMessage` type.
- **`src/bridge/message-bridge.ts`** — Core orchestrator. Routes commands (`/reset`, `/stop`, `/status`, `/help`, `/memory`), manages running tasks per chat (one task at a time per `chatId`), executes Claude queries with streaming card updates, handles image input/output, enforces 1-hour timeout.
- **`src/memory/memory-client.ts`** — Lightweight HTTP client for the MetaMemory server. Used by `/memory` commands (list, search, status) for quick Feishu responses without spawning Claude.
- **`src/claude/executor.ts`** — Wraps `query()` from the Agent SDK as an async generator yielding `SDKMessage`. Configures permissionMode, allowedTools, MCP settings, session resume.
- **`src/claude/stream-processor.ts`** — Transforms the raw SDK message stream into `CardState` objects for display. Tracks tool calls, response text, session ID, cost/duration. Also extracts image file paths and plan file paths from tool outputs.
- **`src/feishu/doc-reader.ts`** — Reads Feishu documents (docx/wiki) and converts Feishu blocks back to Markdown. Reverse of `markdown-to-blocks.ts`. Used by the `lark-doc` skill.
- **`src/claude/session-manager.ts`** — In-memory sessions keyed by `chatId`. Each session has a fixed working directory (from bot config) and Claude session ID. Sessions expire after 24 hours.
- **`src/feishu/card-builder.ts`** — Builds Feishu interactive card JSON. Cards have color-coded headers (blue=thinking/running, green=complete, red=error), tool call lists, markdown response content, and stats (cost/duration). Content truncated at 28KB.
- **`src/feishu/message-sender.ts`** — Feishu API wrapper for sending/updating cards, uploading/downloading images, sending text.
- **`src/bridge/rate-limiter.ts`** — Throttles card updates to avoid Feishu API rate limits (default 1.5s interval). Keeps only the latest pending update.
- **`src/api/peer-manager.ts`** — Manages cross-instance bot discovery and task forwarding. Polls peer MetaBot instances every 30s, caches their bot lists, supports qualified name routing (`peerName/botName`). Anti-loop via `X-MetaBot-Origin` header.
- **`src/web/ws-server.ts`** — WebSocket server for the Web UI. Handles upgrade on `/ws`, token auth via `?token=`, heartbeat, and routes `chat`/`stop`/`answer` messages. Also serves static files from `dist/web/` for the SPA.

### Outputs Directory Pattern

When Claude produces output files (images, PDFs, documents, etc.), they are automatically sent to the user in Feishu:

1. **Per-chat outputs directory** — Before each execution, a fresh directory is created at `/tmp/metabot-outputs/<chatId>/`.
2. **System prompt injection** — Claude is told about the directory via `systemPrompt.append`, instructing it to `cp` output files there.
3. **Post-execution scan** — After execution completes, the bridge scans the directory and sends all files found.
4. **File type routing** — Images (png/jpg/gif/etc.) are sent via `im.v1.image.create`, other files (pdf/docx/zip/etc.) via `im.v1.file.create`.
5. **Size limits** — Images up to 10MB, other files up to 30MB (Feishu API limits).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xvirobotics/metabot](https://github.com/xvirobotics/metabot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
