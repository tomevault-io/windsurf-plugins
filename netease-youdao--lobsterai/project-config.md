---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

```bash
# Development - starts Vite dev server (port 5175) + Electron app with hot reload
npm run electron:dev

# Development with OpenClaw engine (clones/builds OpenClaw on first run)
npm run electron:dev:openclaw

# Build production bundle (TypeScript + Vite)
npm run build

# Lint with ESLint
npm run lint

# Run memory extractor tests (Node.js built-in test runner)
npm run test:memory

# Compile Electron main process only
npm run compile:electron

# Package for distribution (platform-specific)
npm run dist:mac        # macOS (.dmg)
npm run dist:win        # Windows (.exe)
npm run dist:linux      # Linux (.AppImage)

# Build OpenClaw runtime manually
npm run openclaw:runtime:host   # current platform
```

**Requirements**: Node.js >=24 <25. Windows builds require PortableGit (see README.md for setup).

**OpenClaw env vars**: `OPENCLAW_SRC` (default `../openclaw`), `OPENCLAW_FORCE_BUILD=1` (force rebuild), `OPENCLAW_SKIP_ENSURE=1` (skip version checkout).

## Architecture Overview

LobsterAI is an Electron + React desktop application with two primary modes:
1. **Cowork Mode** - AI-assisted coding sessions powered by OpenClaw as the primary agent engine
2. **Artifacts System** - Rich preview of code outputs (HTML, SVG, React, Mermaid)

Uses strict process isolation with IPC communication.

### Authentication Flow

1. **登录：** 打开系统浏览器 → Portal 登录页 → URS 登录成功 → deep link `lobsterai://auth/callback?code=<authCode>`
2. **换取令牌：** `POST /api/auth/exchange` 消费一次性 authCode → 返回 `accessToken`(2h) + `refreshToken`(30d)
3. **持久化：** SQLite kv store `auth_tokens` 存储双 token，应用重启后自动恢复登录态
4. **请求认证：** `fetchWithAuth()` 在每个 API 请求附加 `Authorization: Bearer <accessToken>`
5. **被动刷新：** 收到 HTTP 401 → 使用 refreshToken 调用 `POST /api/auth/refresh` → 获取新 accessToken → 重试原请求
6. **主动刷新：** 定期检查 accessToken 距 exp < 5 分钟 → 后台静默刷新，避免请求失败
7. **滚动续期：** 每次 refresh 签发新 refreshToken（新 30 天有效期），连续使用不掉线
8. **退出条件：** 连续 30 天不使用（refreshToken 过期）→ 清除本地 token → 用户需重新登录

**关键文件：**
- Token 存储与请求：`src/renderer/services/api.ts`（`fetchWithAuth()`、token 管理）
- 登录流程：`src/main/main.ts`（deep link 处理 `lobsterai://` 协议）
- 持久化：`src/main/sqliteStore.ts`（kv 表存储 `auth_tokens`）

### Process Model

**Main Process** (`src/main/main.ts`):
- Window lifecycle management
- SQLite storage via `better-sqlite3` (`src/main/sqliteStore.ts`)
- Agent engine routing (`src/main/libs/agentEngine/coworkEngineRouter.ts`) - dispatches to `openclawRuntimeAdapter.ts` (OpenClaw)
- IM gateways (`src/main/im/`) - WeChat, WeCom, DingTalk, Feishu, QQ, Telegram, Discord, NetEase IM, NetEase Bee, POPO
- Skill management (`src/main/skillManager.ts`)
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
├── im/                  # IM gateway integrations (WeChat/WeCom/DingTalk/Feishu/QQ/Telegram/Discord/POPO)
└── libs/
    ├── agentEngine/
    │   ├── coworkEngineRouter.ts    # Routes to OpenClaw runtime
    │   └── openclawRuntimeAdapter.ts # OpenClaw gateway adapter
    ├── openclawEngineManager.ts # OpenClaw runtime lifecycle (install/start/status)
    ├── openclawConfigSync.ts    # Syncs cowork config → OpenClaw config files

src/renderer/
├── types/cowork.ts      # Cowork type definitions
├── store/slices/
│   ├── coworkSlice.ts   # Cowork sessions and streaming state
│   └── artifactSlice.ts # Artifacts state
├── services/
│   ├── cowork.ts        # Cowork service (IPC wrapper, Redux integration)
│   ├── api.ts           # LLM API with SSE streaming
│   └── artifactParser.ts # Artifact detection and parsing
├── components/
│   ├── cowork/          # Cowork UI components
│   │   ├── CoworkView.tsx          # Main cowork interface
│   │   ├── CoworkSessionList.tsx   # Session sidebar
│   │   ├── CoworkSessionDetail.tsx # Message display
│   │   └── CoworkPermissionModal.tsx # Tool permission UI
│   └── artifacts/       # Artifact renderers

SKILLs/                  # Custom skill definitions for cowork sessions
├── skills.config.json   # Skill enable/order configuration
├── docx/                # Word document generation skill
├── xlsx/                # Excel skill
├── pptx/                # PowerPoint skill
└── ...
```

### Data Flow

1. **Initialization**: `src/renderer/App.tsx` → `coworkService.init()` → loads config/sessions via IPC → sets up stream listeners

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [netease-youdao/LobsterAI](https://github.com/netease-youdao/LobsterAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
