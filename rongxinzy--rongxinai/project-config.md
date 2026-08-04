---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Build and Development Commands

```bash
# Development - starts Vite dev server (port 5175) + Electron app with hot reload
npm run electron:dev

# Development with OpenClaw engine (clones/builds OpenClaw on first run)
npm run electron:dev:openclaw

# Build production bundle (TypeScript + Vite)
npm run build

# Lint with oxlint (config: .oxlintrc.json)
npm run lint

# Format with oxfmt (config: .oxfmtrc.json)
npm run format

# Run unit tests (Vitest)
npm test

# Compile Electron main process only
npm run compile:electron

# Package for distribution (platform-specific)
npm run dist:mac        # macOS (.dmg)
npm run dist:win        # Windows (.exe)
npm run dist:linux      # Linux (.AppImage)

# Build OpenClaw runtime manually
npm run openclaw:runtime:host   # current platform
```

**Requirements**: Node.js >=24 <25, Bun >=1.3 (package manager; `bun install` instead of `npm install`, lockfile is `bun.lock`). Windows builds require PortableGit (see README.md for setup).

**OpenClaw env vars**: `OPENCLAW_SRC` (default `../openclaw`), `OPENCLAW_FORCE_BUILD=1` (force rebuild), `OPENCLAW_SKIP_ENSURE=1` (skip version checkout).

## Architecture Overview

知远智能体 is an Electron + React desktop application for local-first AI Agent workflows. Its core areas are:

1. **Cowork Mode** - AI-assisted task sessions powered by OpenClaw as the primary agent runtime
2. **llama.cpp Local Inference** - local model service management, model launch options, and local model integration with OpenClaw
3. **Skills and MCP** - built-in skills, remote skill marketplace, and MCP server configuration
4. **Artifacts System** - rich preview of code outputs (HTML, SVG, React, Mermaid)

Uses strict process isolation with IPC communication.

Public-facing product documentation and user-visible UI copy must use the 知远智能体 (ZhiYuan Agent) name. All pre-rebrand product names (including 知远智能体, LEO, and 李知远) are retired — do not reintroduce them in branding. OpenClaw, pi, and llama.cpp are internal implementation details: never expose them in branding or user-facing copy; describe the agent runtime and local inference as self-developed (全栈自研). Legacy identifiers (the old storage name, the retired SQLite filename, the old app data directory, the retired protocol scheme, and legacy session keys) have been fully replaced with the new 知远智能体 (ZhiYuan Agent) identifiers under a scorched-earth policy: no data migration, no compatibility shims, no fallbacks; pre-rename user data is abandoned in place. New code must use the new identifiers only.

### Authentication Flow

1. **登录：** 打开系统浏览器 → Portal 登录页 → 登录成功 → deep link callback with `code=<authCode>`
2. **换取令牌：** `POST /api/auth/exchange` 消费一次性 authCode → 返回 `accessToken`(2h) + `refreshToken`(30d)
3. **持久化：** SQLite kv store `auth_tokens` 存储双 token，应用重启后自动恢复登录态
4. **请求认证：** `fetchWithAuth()` 在每个 API 请求附加 `Authorization: Bearer <accessToken>`
5. **被动刷新：** 收到 HTTP 401 → 使用 refreshToken 调用 `POST /api/auth/refresh` → 获取新 accessToken → 重试原请求
6. **主动刷新：** 定期检查 accessToken 距 exp < 5 分钟 → 后台静默刷新，避免请求失败
7. **滚动续期：** 每次 refresh 签发新 refreshToken（新 30 天有效期），连续使用不掉线
8. **退出条件：** 连续 30 天不使用（refreshToken 过期）→ 清除本地 token → 用户需重新登录

**关键文件：**

- Token 存储与请求：`src/renderer/services/api.ts`（`fetchWithAuth()`、token 管理）
- 登录流程：`src/main/main.ts`（deep link callback 处理；legacy protocol names may still be present）
- 持久化：`src/main/sqliteStore.ts`（kv 表存储 `auth_tokens`）

### Process Model

**Main Process** (`src/main/main.ts`):

- Window lifecycle management
- SQLite storage via `better-sqlite3` (`src/main/sqliteStore.ts`)
- Agent engine routing (`src/main/libs/agentEngine/coworkEngineRouter.ts`) - dispatches to `openclawRuntimeAdapter.ts` (OpenClaw)
- llama.cpp lifecycle and local inference management (`src/main/libs/llamacppManager.ts`, `src/shared/llamacpp/`)
- Skill management (`src/main/skillManager.ts`)
- MCP server configuration and marketplace integration
- IM/email gateways (`src/main/im/`) - public-facing channels are WeChat, WeCom, DingTalk, Feishu/Lark, QQ, and Email. Legacy/global connector code may exist; do not re-expose it in UI or docs unless explicitly requested.
- IPC handlers for store, cowork, and API operations (40+ channels)
- Security: context isolation enabled, node integration disabled, sandbox enabled

**Preload Script** (`src/main/preload.ts`):

- Exposes `window.electron` API via `contextBridge`
- Includes `cowork` namespace for session management and streaming events

**Renderer Process** (React in `src/renderer/`):

- All UI and business logic
- Communicates with main process exclusively through IPC

### Key Directories

```
src/main/
├── main.ts              # Entry point, IPC handlers
├── sqliteStore.ts       # SQLite database (kv + cowork tables)
├── coworkStore.ts       # Cowork session/message CRUD operations
├── skillManager.ts      # Skill loading and management
├── im/                  # IM/email gateway integrations
└── libs/
    ├── agentEngine/
    │   ├── coworkEngineRouter.ts    # Routes to OpenClaw runtime
    │   └── openclawRuntimeAdapter.ts # OpenClaw gateway adapter
    ├── openclawEngineManager.ts # OpenClaw runtime lifecycle (install/start/status)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rongxinzy/RongxinAI](https://github.com/rongxinzy/RongxinAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
