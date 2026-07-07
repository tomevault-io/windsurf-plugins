---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

TeamAgentX is a multi-agent collaboration platform with a Feishu/Lark-like UI. Users interact with multiple AI agents by mentioning them with `@agent-name` in chatrooms. Agents can collaborate, hand off tasks, run shell/ACP tools, maintain room memory, create todos, and execute scheduled room tasks.

The product currently includes a React web app, an Electron desktop shell, a Flutter mobile app, and a Fastify backend.

## Architecture

**Monorepo Structure**:
- `apps/web/` - React frontend (Feishu-style UI)
- `apps/desktop/` - Electron desktop shell that packages the web UI and backend
- `apps/mobile/` - Flutter mobile app
- `server/` - Fastify backend with Socket.io, Prisma, agent execution, cron tasks, uploads, and desktop entry
- `docs/` - Project documentation

**Tech Stack**:
- Web: React 19, TypeScript, Vite 6, Tailwind CSS 4, shadcn/ui (new-york style), Socket.io-client, Zustand, react-mentions, react-markdown
- Desktop: Electron 41, Vite, electron-builder, Electron `utilityProcess` for the embedded backend
- Mobile: Flutter/Dart, Provider, go_router, Dio, socket_io_client, WebView, QR scanner
- Backend: Fastify 5, TypeScript, Socket.io, Prisma 7 (SQLite/libsql adapter), LangChain/LangGraph, ACP SDKs, OpenAI Codex SDK, JWT auth

## Common Commands

```bash
# Start web mode from project root (server 3001 + Vite 5173)
./start.sh
./start.sh web

# Start Electron dev mode from project root (embedded server 11053)
./start.sh electron

# Server (from server/)
pnpm dev                  # Development with watch mode
pnpm start                # Run src/index.ts with tsx
pnpm build                # TypeScript build
pnpm db:migrate           # Run Prisma migrations
pnpm db:generate          # Generate Prisma client
pnpm db:seed              # Seed database
pnpm db:studio            # Open Prisma Studio
pnpm test                 # Run node:test with coverage
pnpm test:watch           # Run tests in watch mode

# Web (from apps/web/)
pnpm dev                  # Start Vite dev server
pnpm build                # Production build (tsc -b + vite build)
pnpm lint                 # Run ESLint
pnpm preview              # Preview production build

# Desktop (from apps/desktop/)
pnpm dev                  # Electron dev mode
pnpm typecheck            # Type-check Electron main/preload code
pnpm build                # Build packaged desktop app
pnpm electron:build       # Alias for desktop build

# Mobile (from apps/mobile/)
flutter pub get
flutter run
flutter analyze
flutter test
```

## Desktop DMG Debugging

- 打包后的 macOS 桌面版（DMG / `.app`）启动日志会写入：
  `~/Library/Application Support/@teamagentx/desktop/electron-debug.log`
- 排查 DMG / Electron 打包问题时，先查看这个日志，再看接口报错和数据库状态。
- Electron 内置后端固定监听 `11053`，移动端 Web 入口服务监听 `11054`。
- 常用命令：

```bash
tail -n 200 "~/Library/Application Support/@teamagentx/desktop/electron-debug.log"
tail -f "~/Library/Application Support/@teamagentx/desktop/electron-debug.log"
```

- 桌面版 SQLite 数据库默认路径：
  `~/Library/Application Support/@teamagentx/desktop/teamagentx.db`
- 桌面版上传目录在 userData 下：
  `~/Library/Application Support/@teamagentx/desktop/uploads/images`
- 例如检查 `ChatRoom` 表结构：

```bash
sqlite3 "~/Library/Application Support/@teamagentx/desktop/teamagentx.db" 'PRAGMA table_info("ChatRoom");'
```

## Key Architecture Patterns

### Agent Execution System
- **Factory Pattern**: `server/src/core/agent/executor.factory.ts` creates `LangChainAgentExecutor`, `ClaudeAgentSdkExecutor`, `CodexSdkExecutor`, or generic `AcpExecutor`.
- **Agent Types**: Prisma `Agent.type` is `builtin` or `acp`; ACP agents use `acpTool` such as `claude` or `codex`. The visible ACP tools in the UI are currently Claude and Codex.
- **Interface**: `IAgentExecutor` defines the shared executor contract.
- **Queue Processing**: Agent tasks are queued in `TaskQueue` and processed sequentially per chatroom-agent context.
- **Caching**: Executor instances are cached per chatRoom-agent combination for memory/session isolation. Clear the room cache when room execution settings such as `workDir` change.
- **Work Directory Rules**: Runtime workdir priority is quick-chat/session directory, then `ChatRoom.workDir`, then the default directory. Group assistants no longer have per-room custom work directories. `Agent.workDir` remains the assistant-level config and skills base; `ChatRoomAgent.customWorkDir` is legacy data and should not be used for new runtime behavior.
- **Memory**: `AgentRoomMemory` stores long-term per-room/per-agent summaries; recent messages and compact thresholds are controlled by config env vars.
- **Event-Driven**: `EventEmitter` emits `receivedMessage` events that trigger agent handling.

### Real-Time Communication
- Socket.io with JWT authentication.
- Main room events include `message`, `agent:typing`, `agent:stream`, `agent:thinking`, `agent:tool_call`, `agent:done`, `agent:status`, `agent:task-queue`, `agent:task-cancelled`, `agent:task-resumed`, and `unread:update`.
- Rooms are scoped by `chatRoomId`; user-specific pushes use `user:<userId>` rooms.

### Database Models

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dbfu/teamagentx](https://github.com/dbfu/teamagentx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
