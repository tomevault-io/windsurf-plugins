---
trigger: always_on
description: > Bridge WeChat direct messages to OpenCode Server via HTTP API.
---

# AGENTS.md — wechat-opencode

> Bridge WeChat direct messages to OpenCode Server via HTTP API.

## Project Overview

- **Package**: `wechat-bridge-opencode` v1.3.7 — ESM-only (`"type": "module"`)
- **Runtime**: Node.js 20+
- **Language**: TypeScript, compiled to JS via `tsc`
- **Package manager**: npm (use `package-lock.json`)
- **Repository**: https://github.com/pan17/wechat-opencode

## Commands

```bash
npm install          # Install dependencies
npm run build        # Compile TypeScript → dist/
npm run dev          # Watch mode: tsc --watch
npm start            # Run compiled CLI: node dist/bin/wechat-opencode.js
npm run prepack      # Runs build before npm publish
npm test             # Run vitest unit tests (278 tests across 14 files)
npm run test:watch   # Vitest in watch mode
```

**Tests:** Vitest 4.1.8 unit tests live in `src/__tests__/` (278 tests across 14 files). No linter is configured.

### Running the CLI locally

```bash
npm run build
node dist/bin/wechat-opencode.js --help
node dist/bin/wechat-opencode.js          # auto-starts opencode serve
node dist/bin/wechat-opencode.js --server-url http://localhost:4096  # uses external server
```

## Architecture

```
bin/wechat-opencode.ts          — CLI entry (arg parsing, daemon, sidecar server, QR rendering)
src/index.ts                    — Public API exports
src/bridge.ts                   — Main orchestrator (WeChat poll ↔ OpenCode Server HTTP)
src/config.ts                   — Config types, defaults, server connection config
src/types.ts                    — Shared types (MessagePart, ModelRef, MediaContent)
src/vendor.d.ts                 — Type declarations for untyped npm packages
src/types/
  events.ts                     — SSE event types (message.*, session.*, question.*, permission.*)
  question.ts                   — Question tool types (QuestionPrompt, PendingQuestion, …)
  permission.ts                 — Permission tool types (PermissionRequest, PendingPermission, AutoPermissionMode, …)
src/server/
  client.ts                     — OpenCode Server HTTP client (fetch wrapper)
  session.ts                    — Simplified SessionManager (no subprocess, just HTTP)
  event-pipeline.ts             — Persistent /global/event SSE connection with reconnect
src/__tests__/                  — Vitest unit tests (14 files, 278 tests)
src/adapter/
  inbound.ts                    — WeChat message → MessagePart[] (text, image, file)
  outbound.ts                   — Server reply → WeChat text (formatting, splitting)
  workspace-cmd.ts              — Parse /workspace, /session, /agent, /model, /reasoning, /help, /reject-question, /reject-permission, /auto-permission commands
  question-format.ts            — Format & parse question replies (Q{n}={value} / Q{n}-{value} grammar)
  permission-format.ts          — Format & parse permission replies (1/2/3 / P{n}={once|always|reject} grammar)
  thinking-format.ts            — Reasoning summary + tool summary formatting
src/weixin/
  auth.ts                       — WeChat iLink login (QR code, token persistence)
  monitor.ts                    — Long-poll for new messages
  send.ts                       — Send text/image/file/video to WeChat
  api.ts                        — WeChat iLink API (typing indicator, config)
  media.ts                      — CDN download + AES decryption
  types.ts                      — WeChat iLink types (MessageType, UploadMediaType, etc.)
```

### Key flows
1. **CLI** starts `opencode serve` as sidecar → creates `WeChatOpencodeBridge`
2. **Bridge** handles QR login → creates `SessionManager` (HTTP client) → begins WeChat long-poll
3. **SessionManager** communicates with OpenCode Server via HTTP REST API (POST /session, POST /session/:id/message)
4. **Adapters** convert WeChat messages ↔ server message parts

### Session management
- **Single-user**: no Map-based routing, no per-user subprocess
- Agent process is managed by `opencode serve` — bridge only sends HTTP requests
- Session ID is persisted in `~/.wechat-opencode/.wechat-bridge-state.json`
- Mode/model/reasoning are passed as per-request parameters, not ACP RPC calls
- ACL, permission handling, tool execution are all server-side

## Code Style

### Imports
- **Always use `.js` extension** in relative imports (ESM requirement):
  ```ts
  import { WeChatOpencodeBridge } from "./bridge.js";
  ```
- **Node built-ins** use `node:` prefix:
  ```ts
  import fs from "node:fs";
  import path from "node:path";
  ```
- Group order: Node built-ins → npm packages → relative imports
- Prefer **named exports** over default exports (only `qrcode-terminal` uses default)

### TypeScript
- **Strict mode** enabled (`"strict": true` in tsconfig)
- **Target**: ES2022, **Module**: NodeNext, **ModuleResolution**: NodeNext
- Use `interface` for object shapes/config types, `type` for unions and derived types
- **No `as any`**, `@ts-ignore`, or `@ts-expect-error`
- Declaration files: `declaration: true`, `declarationMap: true`

### Naming
- **Classes**: `PascalCase` — `WeChatOpencodeBridge`, `SessionManager`
- **Interfaces**: `PascalCase` — `WeChatOpencodeConfig`, `SessionMode`
- **Functions/methods**: `camelCase` — `handleMessage`, `sendReply`
- **Constants**: `UPPER_SNAKE_CASE` — `TEXT_CHUNK_LIMIT`, `MSG_LIMIT_MAX`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pan17/wechat-bridge-opencode](https://github.com/pan17/wechat-bridge-opencode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
