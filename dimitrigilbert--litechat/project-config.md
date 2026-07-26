---
trigger: always_on
description: This file provides guidance to opencode when working with code in this repository.
---

# AGENT.md

This file provides guidance to opencode when working with code in this repository.

## LiteChat Development Commands

### Development
```bash
npm run build              # Build for production with TypeScript compilation
npm run build:en           # Build English version specifically
npm run build:fr           # Build French version specifically
npm run build:all          # Build all language versions
```

### Deployment & Services
```bash
npm run deploy             # Deploy to GitHub Pages (gh-pages -d dist)
npm run to2web             # Build, release, and deploy to web
npm run mcp-proxy          # Start MCP bridge service (node bin/mcp-bridge.js)
npm run serve              # Build and serve with http-server on LITECHAT_PORT (default: 5173)
npm run update             # Git pull, npm install, and serve (respects LITECHAT_ORIGIN and LITECHAT_BRANCH)
```

**⚠️ CRITICAL: NEVER run `npm run serve` or start any servers - user has their own dev environment running!**
**⚠️ CRITICAL: NO DYNAMIC IMPORTS UNLESS EXPRESSLY TOLD SO ! NEVER !**

### Docker
```bash
# Manual build
npm run build && docker build -t litechat .
docker run -d -p 8080:3000 litechat

# Docker Compose (includes MCP bridge)
docker-compose up -d
```

## Architecture Overview

LiteChat is a 100% client-side AI chat application built with a modular, event-driven architecture designed for extensibility and privacy.

### Core Technology Stack
- **Frontend**: React 19, TypeScript, Vite, Tailwind CSS, shadcn/ui
- **State Management**: Zustand with Immer middleware and domain-specific stores
- **Data Storage**: Dexie.js (IndexedDB) with ZenFS for virtual file system
- **AI Integration**: Vercel AI SDK supporting multiple providers (OpenAI, Claude, Gemini, OpenRouter, local models)
- **Version Control**: isomorphic-git for browser-based Git operations
- **Internationalization**: i18next with React integration

### Modular Control System

LiteChat uses a sophisticated control module architecture with three scopes:

1. **PromptControls**: Input area extensions and prompt manipulation
2. **ChatControls**: Sidebar panels, headers, and general UI controls  
3. **CanvasControls**: Action buttons and interactions within chat canvas

**Control Module Lifecycle**:
- `initialize()`: Setup phase
- `register(modApi)`: Registration with mod API
- `destroy()`: Cleanup and resource deallocation

**Key Directories**:
- `src/controls/modules/`: Control module implementations
- `src/controls/components/`: UI components for controls
- `src/store/control.store.ts`: Central control registry

### Event-Driven Architecture

All inter-system communication uses a centralized event emitter with strongly-typed events:

- **Event Types**: Organized in `src/types/litechat/events/` by domain
- **Event Coordinator**: `EventActionCoordinatorService` automatically binds store actions to events
- **Store Integration**: Stores emit events on state changes and expose `getRegisteredActionHandlers()`

### State Management Pattern

Domain-specific Zustand stores with consistent patterns:
- **Immer middleware**: Immutable state updates
- **Event integration**: Automatic event emission on changes
- **Action handlers**: Exposed for event coordinator registration
- **Type safety**: Full TypeScript coverage

**Key Stores**:
- `conversation.store.ts`: Chat data and state
- `settings.store.ts`: Application configuration
- `provider.store.ts`: AI provider management
- `vfs.store.ts`: Virtual file system state

### Block Renderer System

Extensible content rendering with priority-based selection:

- **Language-specific renderers**: Handle code blocks, diagrams, etc.
- **Universal fallback**: Default renderer with enhanced features
- **Context-aware**: Rich context including streaming state, file paths
- **Registration**: `src/services/block-renderer.service.ts`

**Example Renderers**:
- `JsRunnableBlockRenderer`: JavaScript execution with safe/unsafe/iframe modes
- `MermaidBlockRenderer`: Real-time diagram rendering
- `PythonRunnableBlockRenderer`: Python code execution

### Virtual File System (VFS)

Browser-based filesystem using ZenFS + IndexedDB:
- **Full CRUD operations**: Create, read, update, delete files/directories
- **Git integration**: Clone, commit, push, pull repositories
- **Project organization**: Hierarchical project structure
- **Sync capabilities**: Git-based conversation synchronization

### Modding API

Safe, controlled extension interface:
- **Resource management**: Automatic cleanup on mod unload
- **Sandboxed API**: `createModApi()` provides controlled access
- **Event system access**: Mods can emit/listen to typed events
- **Tool registration**: Dynamic AI tool addition

### Build System Features

- **Build-time configuration**: Custom system prompts via `VITE_SYSTEM_PROMPT_FILE`
- **Multi-language builds**: Automatic language detection and building
- **Node.js polyfills**: Browser compatibility for Node.js APIs
- **PWA support**: Service worker and offline capabilities

### Security & Privacy

- **100% client-side**: No server dependencies for core functionality
- **Local storage**: All data in browser IndexedDB
- **Code execution**: Multiple isolation levels (QuickJS VM, iframe sandbox, direct eval)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DimitriGilbert/LiteChat](https://github.com/DimitriGilbert/LiteChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
