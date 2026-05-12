---
trigger: always_on
description: ClawCenter is a TypeScript central router/dispatcher bridging multiple WeChat accounts
---

# AGENTS.md - ClawCenter (dispatch)

## Project Overview

ClawCenter is a TypeScript central router/dispatcher bridging multiple WeChat accounts
to multiple AI agents (Claude Code, OpenCode, CodeBuddy, Codex, etc.). It has two
operational modes: **center** (dispatcher + WeChat connector + worker hub) and **worker**
(connects to a center, runs local agents). The codebase includes a Node.js backend
(`src/`) and a React web management panel (`web/`).

## Build / Run / Test Commands

| Command | Description |
|---------|-------------|
| `npm run build` | Full build: compile server (`tsc`) + build web frontend (Vite) |
| `npm run build:server` | Server only: `tsc` |
| `npm run build:web` | Web frontend only: `vite build --config web/vite.config.ts` |
| `npm run typecheck` | Type-check without emitting: `tsc --noEmit` |
| `npm run dev` | Dev server with hot reload: `tsx src/index.ts start` |
| `npm run dev:worker` | Dev worker mode: `tsx src/index.ts start --worker` |
| `npm run start` | Production: `node dist/index.js start` |

**Testing:** No test framework is configured. There are no test files or test runner.
If you add tests, Vitest is the recommended choice given the Vite-based web frontend.

**Linting/Formatting:** No linter or formatter is configured. Follow the conventions
described below to maintain consistency.

**Always run `npm run typecheck` after making changes** to verify there are no type errors.

## Tech Stack

- **Runtime:** Node.js >= 22, ESM modules (`"type": "module"`)
- **Language:** TypeScript with strict mode enabled
- **Server:** Fastify, Commander (CLI), ws (WebSocket)
- **Frontend:** React 19 + Vite + React Router DOM
- **Database:** sql.js (SQLite in-process WASM)
- **Validation:** Zod
- **TUI:** Ink + React (terminal UI)
- **AI SDKs:** `@anthropic-ai/claude-agent-sdk`

## Project Structure

```
src/
  index.ts              # CLI entry (Commander)
  server.ts             # Fastify server bootstrap
  logger.ts             # Logging utilities
  core/
    agents/             # Agent adapters (one file per agent type)
      adapter.ts        # AgentAdapter interface + shared types
      manager.ts        # Agent lifecycle management
    api/routes.ts       # All REST API route definitions
    db/
      schema.ts         # SQLite schema + defaults
      store.ts          # Data access layer (all CRUD)
  center/
    dispatcher.ts       # Message dispatch orchestration
    hub.ts              # WebSocket server for remote workers
    router/             # Routing engine + hashtag parser
    commands/           # System slash commands
    wechat/             # WeChat API client, connector, login, CDN
  worker/client.ts      # Worker WebSocket client
  tui/                  # Terminal UI (Ink/React)
web/                    # Separate Vite sub-project (management panel)
  src/
    pages/              # One component per page (Dashboard, Settings, etc.)
```

## Code Style Guidelines

### Formatting

- **Indentation:** 2 spaces (no tabs)
- **Quotes:** Double quotes `"` everywhere (strings, imports, JSX attributes)
- **Semicolons:** Always required
- **Trailing commas:** Yes, in multi-line constructs (function params, objects, arrays)
- **Braces:** Same-line opening brace (K&R style)
- **Line length:** Aim for ~100-120 characters; no strict limit enforced

### Imports

- Use **ESM imports** with `.js` extension on all local imports (required for ESM):
  ```typescript
  import { Store } from "./core/db/store.js";
  ```
- **Ordering** (not strictly enforced, but preferred):
  1. Node.js built-ins (`node:path`, `node:fs`, `node:crypto`)
  2. Third-party packages (`fastify`, `zod`, `ws`)
  3. Local/relative imports (`./core/db/store.js`)
- Use `import type` or inline `type` keyword for type-only imports:
  ```typescript
  import type { FastifyInstance } from "fastify";
  import { Store, type Database } from "./core/db/store.js";
  ```

### Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Files (non-component) | kebab-case | `claude-code.ts`, `system-commands.ts` |
| React component files | PascalCase | `App.tsx`, `Dashboard.tsx` |
| Classes | PascalCase | `AgentManager`, `WechatConnector` |
| Interfaces / Types | PascalCase | `ServerOptions`, `AgentConfig` |
| Functions / variables | camelCase | `startServer`, `agentManager` |
| Module-level constants | UPPER_SNAKE_CASE | `MAX_CONSECUTIVE_FAILURES`, `SCHEMA_SQL` |
| Boolean variables | `is`/`has` prefix | `isRunning`, `hasDefault` |
| Database columns | snake_case | `wechat_id`, `created_at` |
| Private class members | `private` keyword, no prefix | `private db`, `private running` |

### Types

- **Strict mode is enabled** -- do not add `// @ts-ignore` or `// @ts-nocheck`.
- Prefer `interface` for object shapes, `type` for unions and function signatures:
  ```typescript
  interface AgentConfig { model?: string; workDir?: string; }
  type AgentType = "claude-code" | "claude-sdk" | "opencode";
  type WorkerSendFn = (agentId: string, params: SendParams) => Promise<SendResult>;
  ```
- **Avoid `any`** -- use `unknown` for truly unknown values, `Record<string, unknown>`
  for untyped objects. The codebase has ~10 `any` usages, all for parsing untyped
  external JSON.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ruihanglix/clawcenter](https://github.com/ruihanglix/clawcenter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
