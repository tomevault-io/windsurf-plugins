---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project

Golemancy — AI Agent orchestration platform. Electron desktop app with pixel art (Minecraft) aesthetic, dark theme only. Language: Chinese for discussions/docs, English for code.

GitHub: https://github.com/jicaiinc/golemancy | Domain: golemancy.ai | Copyright: Jicai, Inc. | Website repo: `/Users/cai/developer/github/golemancyweb`

## Rules

**绝对禁令：**
- **永远不要动 git** — 可以查看，但绝对不允许提交代码
- **Plan mode 用中文**

**Fact-Based Analysis**: 需要分析时，必须查阅实际证据（官方文档、WebSearch、Context7、源码），不要依赖训练知识。

**Agent Model Preferences** (spawning via Agent tool):
- `model: "opus"` — complex tasks (refactoring, features, debugging)
- sonnet — moderate tasks (code review, tests, straightforward edits)
- `model: "haiku"` — **only** for simple file search/grep (Explore agent)
- When in doubt, prefer stronger model

**`__guidelines/` 目录只读** — 未经用户允许不得新增、修改或删除。每个主题独立子文件夹，格式 `{topic}-{YYYYMMDD}`。当前内容：`__guidelines/i18n-20260302/`（翻译基准 + 开发规范）。

### Critical Library Choices

Do NOT use the alternatives:

| Use this | NOT this | Why |
|----------|----------|-----|
| `motion/react` | `framer-motion` | motion is the current package |
| `react-router` | `react-router-dom` | v7 unified package |
| `@tailwindcss/postcss` | `@tailwindcss/vite` | vite plugin bugs with electron-vite dev |
| Tailwind CSS v4 CSS-first (`@theme {}` in global.css) | `tailwind.config.js` | v4 architecture |
| `hono` | `express` | Server HTTP framework |
| `drizzle-orm` | `prisma` / raw SQL | Server ORM |
| `better-sqlite3` | `sql.js` / other | Server database |
| `ai` (Vercel AI SDK v6) | direct API calls | AI integration |
| `react-i18next` + `i18next` | `react-intl` / `lingui` | i18n framework |

## Commands

```bash
pnpm dev                    # Development (Electron + server + HMR)
pnpm build                  # Build all packages
pnpm lint                   # Type-check all packages
pnpm test                   # Run all tests
pnpm test:live              # Live tests (requires running server + API keys)
pnpm test:build             # Build preflight checks
pnpm pack                   # Package for local test
pnpm dist                   # Package for distribution

# Single package
pnpm --filter @golemancy/ui test
pnpm --filter @golemancy/server test
pnpm --filter @golemancy/server dev              # Server standalone

# Single test file
pnpm --filter @golemancy/ui exec vitest run src/components/base/PixelButton.test.tsx
pnpm --filter @golemancy/ui exec vitest src/components/base/PixelButton.test.tsx  # watch mode
```

## Monorepo Structure

```
apps/desktop/      @golemancy/desktop  — Electron shell, forks server as child process
packages/ui/       @golemancy/ui       — React UI, business logic, store, services
packages/server/   @golemancy/server   — Hono HTTP server, SQLite, AI agent runtime
packages/shared/   @golemancy/shared   — Pure TypeScript types + service interfaces (zero runtime)
packages/tools/    @golemancy/tools    — Browser automation tool (Playwright-based)
```

Dependency: `desktop → ui → shared ← server ← tools` (strict one-way). Turborepo + pnpm v10 workspaces.

## Architecture

### Core Abstractions

Four top-level abstractions: **Project** (container), **Agent** (core unit), **Team** (agent topology), **Memory** (agent-scoped knowledge). All agents belong to a project. Each project has a Main Agent (`defaultAgentId`) and optionally a Default Team (`defaultTeamId`). Projects also have Cron Jobs for scheduled execution. No global Agent/Skill libraries.

**Agent capabilities** assembled at runtime (`agent/tools.ts`):
- **Skills** — `agent.skillIds` → project-scoped, injected into system prompt
- **MCP** — `agent.mcpServers` → connects to MCP servers, loads their tools
- **Built-in Tools** — `agent.builtinTools: { bash?, browser?, computer_use?, task?, memory? }` → permission mode controls execution
- **Memory** — Agent-scoped, persists in SQLite. Pinned memories always load; non-pinned load top-N by priority + recency
- **Sub-Agents** — Agent itself has NO sub-agent fields. When conversation has `teamId`, runtime creates `delegate_to_{agentId}` tools from `TeamMember[]` children. Lazy-loaded, infinite nesting

**System prompt** = `agent.systemPrompt` + skill instructions + memory context + team instruction (injected as `## Team Context`)

**Team** = single-parent tree of `TeamMember { agentId, role, parentAgentId? }`. Agents decoupled from Teams (same agent can join multiple teams). Conversation scoped to Team via `conversation.teamId` activates sub-agent delegation; CronJob also supports `teamId`. Details in `_team/team.md`.

### Electron ↔ Server

Electron main fork()s server child process (PORT=0), server sends `{ port, token }` via IPC, main passes to renderer via `additionalArguments` + preload `contextBridge`. All UI↔Server communication is HTTP to `127.0.0.1:port` with per-session Bearer token. Without Electron, UI falls back to mock services.

**Pitfalls**: see `_pitfalls/electron-server-fork.md`. Key: use `app.getAppPath()` not `__dirname`; use `execPath: 'node'` in dev for native modules; always `pnpm dev` smoke test after Electron/server changes.

### Server


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jicaiinc/golemancy](https://github.com/jicaiinc/golemancy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
