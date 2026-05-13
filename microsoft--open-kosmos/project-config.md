---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Building and Running
```bash
# Development server (renderer only)
npm run dev

# Full development (main + renderer watch)
npm run dev:full

# Build (main + renderer)
npm run build
npm run build:main       # Build main process only
npm run build:renderer   # Build renderer process only

# Run the application
npm run electron         # After building
npm run electron:dev     # Development mode with dev renderer
npm run start            # Build and run in production mode
npm run start:prod       # Production mode
```

### Testing and Quality
```bash
npm test                 # Run Jest tests
npm run lint             # Check code style
npm run lint:fix         # Auto-fix linting issues
npm run test:build       # Test build integrity
npm run test:build:verify # Verify build output
```

### Building Installers
```bash
npm run dist             # Build for current platform
npm run dist:win         # Windows installer
npm run dist:mac         # macOS DMG
npm run dist:linux       # Linux AppImage
npm run dist:all         # All platforms

# Architecture-specific builds
npm run dist:win:x64
npm run dist:win:arm64
npm run dist:mac:x64
npm run dist:mac:arm64
npm run dist:mac:universal
```

### Release Management
```bash
npm run prepare:release         # Interactive release preparation
npm run prepare:release:patch   # Prepare patch release (x.x.1)
npm run prepare:release:minor   # Prepare minor release (x.1.0)
npm run prepare:release:major   # Prepare major release (1.0.0)

npm run dist:publish            # Build and publish to GitHub releases
npm run dist:publish:win        # Publish Windows build
npm run dist:publish:mac        # Publish macOS build
```

## Architecture Overview

OpenKosmos is an Electron-based AI assistant application (v1.21.7) with a modern React frontend. It supports multi-brand deployment via `BRAND` env variable. The architecture follows Electron's multi-process model with clear separation of concerns.

### Process Architecture

**Main Process** (`src/main/`)
- Handles system-level operations, file I/O, and native integrations
- Manages authentication via GitHub Copilot OAuth device flow
- Controls MCP (Model Context Protocol) server lifecycle
- Persists user data and chat sessions to local storage
- Manages voice features (STT), screenshot capture, browser control, and analytics
- Entry point: `src/main/bootstrap.ts` → `src/main/main.ts`
- `bootstrap.ts` configures brand-specific `userData` path isolation before any module initialization

**Renderer Process** (`src/renderer/`)
- React 18 UI with TypeScript and TailwindCSS
- Two independent entry points: main window (`index.tsx`), screenshot overlay (`screenshot.tsx`)
- Communicates with main process via type-safe Electron IPC (see `src/shared/ipc/`)
- Custom atom-based state management library (`src/renderer/atom/`)
- Entry point: `src/renderer/index.tsx`, main component: `src/renderer/App.tsx`

**Preload Scripts** (`src/main/preload.ts`, `preload-screenshot/entry.ts`)
- Bridge main and renderer processes securely via `contextBridge`
- Each window type has its own preload with scoped API surface
- Type-safe IPC channel whitelisting enforced at compile time via `src/shared/ipc/base.ts`

### Core Systems

#### 1. Authentication System (`src/main/lib/auth/`)
- **MainAuthManager**: Singleton managing authentication state, token refresh, and profile persistence
- **GhcAuthManager**: GitHub Copilot OAuth device flow implementation
- **TokenMonitor**: Monitors token expiration and triggers automatic refresh
- **RefreshTokenAnalyzer**: Validates and analyzes token health
- Authentication flow uses GitHub OAuth device code flow with automatic token refresh
- Stores auth data in `{userData}/profiles/{userAlias}/auth.json`

#### 2. Profile & Data Management (`src/main/lib/userDataADO/`)
- **ProfileCacheManager**: Centralized data management (~110KB) for user profiles, chat configs, agents, skills, and MCP servers
- **ChatSessionManager** / **ChatSessionFileOps**: Handles chat session persistence to individual JSON files
- **AgentAssetsImporter**: Imports agent configurations from external sources
- **AppCacheManager**: Application-level cache (runtime config, feature flags)
- Profiles stored in `{userData}/profiles/{userAlias}/profile.json`
- Chat sessions stored in `{userData}/profiles/{userAlias}/chatSessions/{sessionId}.json`
- Skills stored in `{userData}/profiles/{userAlias}/skills/{skill-name}/`
- Uses in-memory caching with batch notification to frontend (500ms debounce)

#### 3. Chat Engine (`src/main/lib/chat/`)
- **AgentChat**: Core conversation engine (~163KB) — one instance per active chat tab, manages multi-step agent conversations with tool execution
- **AgentChatManager**: Manages AgentChat instance lifecycle (~51KB)
- **GlobalSystemPrompt**: Injects global system instructions into all conversations
- Chat status flow: `IDLE → SENDING_RESPONSE → COMPRESSING_CONTEXT → COMPRESSED_CONTEXT → RECEIVED_RESPONSE`
- Supports `{{KOSMOS_*}}` placeholder substitution in system prompts
- Integrates with memory system for semantic context enhancement

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/open-kosmos](https://github.com/microsoft/open-kosmos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
