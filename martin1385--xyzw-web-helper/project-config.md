---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Vue 3 Token Manager application for XYZW game automation. The application manages game tokens via Base64 decoding, establishes WebSocket connections, and provides a visual interface for token management and game automation.

## Development Commands

### Core Commands
```bash
# Development server (port 3000)
npm run dev
# or
pnpm run dev

# Production build
npm run build

# Preview production build
npm run preview

# Lint Vue, JS, TS files with auto-fix
npm run lint

# Format code (Prettier)
npm run format

# Test individual modules
npm run testr  # Test role token
npm run testd  # Test token
```

### Installation
```bash
# Recommended: use pnpm (as specified in packageManager field)
pnpm install

# Alternative: use npm
npm install
```

### Package Manager Note
This project uses `pnpm@10.19.0` as specified in package.json. While npm works, pnpm is the recommended package manager for consistency.

## Architecture Overview

### Core System Design
The application is built around a **token-centric architecture** that replaces traditional user authentication:

1. **Token Management System**: Base64-encoded tokens are imported, decoded, and stored locally
2. **WebSocket Connection Layer**: Automatic WebSocket connections using BON protocol for game communication
3. **Local-First Storage**: All data stored in browser localStorage and IndexedDB, no backend dependencies
4. **Protocol Layer**: Custom BON (Binary Object Notation) protocol for game message encoding/decoding

### Key Architectural Components

#### 1. Token Store (`src/stores/tokenStore.ts`)
Central state management for token operations using Pinia and VueUse:
- **Token Lifecycle**: Import → Parse → Store → Select → Connect
- **Base64 Parsing**: Supports multiple formats (JSON, plain text, prefixed, bin files, URL sources)
- **WebSocket Management**: Automatic connection establishment with status tracking and connection pooling
- **Connection Queue System**: Rate limiting with configurable max concurrent connections (default 10) and delay intervals
- **Task Coordination**: Tracks running tasks, scheduled tasks, and manages connection lifecycle around task execution
- **Data Persistence**: localStorage via `useLocalStorage` composable with cross-session state recovery
- **TypeScript Interfaces**: `TokenData`, `WebSocketConnection`, `ConnectLock` for type safety

#### 2. BON Protocol Implementation (`src/utils/bonProtocol.js`)
Custom binary protocol for game communication:
- **Message Encoding/Decoding**: Binary serialization with `DataReader`/`DataWriter` classes
- **Type System**: Support for primitives, arrays, maps, nested objects via `bon.encode()/decode()`
- **Encryption Layer**: Multi-channel encryption (LZ4 compression, XOR, XXTEA) with auto-detection
- **Game Message Templates**: `ProtoMsg` class and `GameMessages` helpers for common operations
- **WebSocket Message Handling**: `g_utils` utilities for message parsing and creation

#### 3. WebSocket Client (`src/utils/xyzwWebSocket.js`)
Enhanced WebSocket client with game-specific features:
- **Command Registry**: Pre-registered game commands with default parameters via `CommandRegistry` class
- **Queue Management**: `p-queue` based message queuing with automatic batch processing
- **Connection Management**: Auto-reconnection with exponential backoff, heartbeat system, status monitoring
- **Promise Support**: Both fire-and-forget (`send()`) and request-response (`sendWithPromise()`) patterns
- **Built-in Game Commands**: `getRoleInfo()`, `signIn()`, `claimDailyReward()`, etc.
- **Event Emitter**: Event-driven architecture for connection lifecycle and message handling

#### 4. Router Architecture (`src/router/index.js`)
Token-aware navigation system with file-based routing:
- **File-Based Routes**: Auto-generated from `src/views/` via `unplugin-vue-router`
- **Manual Routes**: Custom routes for Home, TokenImport with query param support
- **Access Control**: Route guards based on token availability (`meta.requiresToken`)
- **Smart Redirects**: Automatic routing based on token state
- **Layout System**: `DefaultLayout.vue` wrapper for admin pages with nested routing

#### 5. Daily Task Automation (`src/utils/dailyTaskRunner.js`)
Task orchestration system for automated game operations:
- **DailyTaskRunner Class**: Centralized task execution with configurable delays
- **Promise-based Commands**: Timeout-aware game command execution via `executeGameCommand()`
- **Task Chaining**: Sequential task execution with error handling and logging
- **Progress Callbacks**: Real-time task progress updates via callback system
- **Common Game Operations**: Sign-in, arena battles, tower climbing, daily rewards

#### 6. Theme System (`src/composables/useTheme.js`)
Reactive dark/light theme management:
- **Global Reactive State**: Shared `isDark` ref across all components
- **DOM Synchronization**: `MutationObserver` based state sync with HTML/body attributes
- **System Theme Detection**: Auto-detect and follow system theme preferences
- **Persistent Preferences**: localStorage backed theme selection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [martin1385/xyzw-web-helper](https://github.com/martin1385/xyzw-web-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
