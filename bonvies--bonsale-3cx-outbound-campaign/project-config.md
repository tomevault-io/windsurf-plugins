---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Commands

### Development
```bash
pnpm install              # Install all dependencies
pnpm dev                  # Run backend & frontend in dev mode
pnpm backend              # Run backend only (Node.js on port 4020)
pnpm frontend             # Run frontend only (Vite on port 4030)
pnpm type-check           # TypeScript validation across all packages
pnpm lint                 # ESLint check
pnpm lint:fix             # ESLint with auto-fix
```

### Docker (Local Development)
```bash
pnpm docker:up            # Start docker-compose (backend + frontend + redis)
pnpm docker:down          # Stop docker-compose
docker-compose logs -f    # View live logs
```

### Build & Test
```bash
pnpm build                # Build all packages (Turbo orchestrated)
pnpm test                 # Run tests
pnpm clean                # Clean build artifacts
```

### Single Package Commands
```bash
# Backend (apps/backend/)
pnpm -F backend dev       # Backend dev server with auto-reload
pnpm -F backend build     # Compile TypeScript to dist/

# Frontend (apps/frontend/)
pnpm -F frontend dev      # Frontend dev with HMR
pnpm -F frontend build    # Production build
```

## Project Architecture

### Technology Stack
- **Frontend**: React 19 + Vite + TypeScript + Material-UI 7
- **Backend**: Node.js 18+ + Express + TypeScript + WebSocket (ws)
- **Data**: Redis 7 (state & cache) + async-mutex (concurrency)
- **Build**: Turbo monorepo + pnpm workspaces
- **Deployment**: Docker + Docker Compose + GCP Container Registry

### Monorepo Structure
```
apps/backend/           # Express API server
  - src/class/         # Core business logic (Project, ProjectManager, etc.)
  - src/services/      # Redis, API integrations (3CX, Bonsale, xApi)
  - src/routes/        # HTTP endpoints
  - src/websockets/    # WebSocket connection handlers
  - src/shared/        # Backend-only shared utilities (alias: @shared-local/*)
  - src/features/
    - outbound-campaign/   # 3CX 自動外播功能
    - call-schedule/       # 自動語音通知功能
      - services/api/      # Phone API 抽象層（IPhoneApiService）
        - device/          # NewRock / Yeastar 實作
      - services/monitor/  # Call Monitor 抽象層（ICallMonitorService）
        - device/          # NewRock / Yeastar 監控實作
      - services/          # callScheduleService, database
      - routes/            # HTTP 路由
      - types/             # 型別定義

apps/frontend/         # React web dashboard
  - src/components/    # React UI components
  - src/pages/         # Page-level components
  - src/router/        # React Router setup
  - src/theme/         # Material-UI theme

packages/shared-types/ # Shared TypeScript definitions
  - ApiResponse<T>, User, Auth types
  - WebSocket event types
```

### High-Level Architecture

This is a **real-time outbound campaign management system** integrating:
1. **3CX Phone System**: WebSocket connection for live call monitoring/control
2. **Bonsale CRM**: API integration for contact management and configurations
3. **Campaign Dashboard**: Real-time UI for viewing/controlling active campaigns

**Key Pattern**: The `Project` class (src/class/project.ts) is the heart of the system, managing a single campaign's full lifecycle:
- 3CX WebSocket connection (connect → authenticate → listen for calls)
- Call list processing and dialing
- Call state tracking (ringing → connected → completed)
- Bonsale CRM updates on call completion
- Automatic recovery on connection loss

The backend exposes this via WebSocket events, allowing the frontend to view/control projects in real-time.

### State Management
- **In-Memory**: Active Project instances maintain state during operation
- **Redis**: Persistent storage of project state (survives restarts with `AUTO_RECOVER_ON_RESTART=true`)
- **Bonsale**: CRM source of truth for contacts and call outcomes

## Core Components to Know

### Backend

**`Project` class** (`src/class/project.ts`, ~95KB)
- Manages a single outbound campaign lifecycle
- Connects to 3CX, monitors calls, dials contacts, updates CRM
- Complex state machine with concurrency control (mutex)
- Implements: token refresh, call restrictions (time/extension), idle checks, auto-recovery

**`ProjectManager`** (`src/class/projectManager.ts`)
- Registry of active Project instances
- Lifecycle operations: create, stop, remove, recover from Redis

**`WebSocketManager`** (`src/class/webSocketManager.ts`)
- Manages WebSocket server connections and message broadcasting
- Two endpoints: main WS server + Bonsale webhook listener

**`redis.ts`** (`src/services/redis.ts`)
- Redis client initialization and cache operations
- Persistence layer for project state

**API Services** (`src/services/api/`)
- `bonsale.ts`: Contact retrieval, campaign config
- `callControl.ts`: 3CX call operations (dial, hangup, transfer)
- `xApi.ts`: Token refresh from 3CX authentication

**Call Schedule Feature** (`src/features/call-schedule/`)
- **`phoneApiService.ts`**: `IPhoneApiService` 抽象層，根據 `TELEPHONE_EQUIPMENT` 環境變數切換設備
  - `device/newRockApi.ts`: NewRock HTTP 1.0 XML API 實作
  - `device/yeastarApi.ts`: Yeastar REST API + 自動 Token 刷新（斷網恢復）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bonvies) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-15 -->
