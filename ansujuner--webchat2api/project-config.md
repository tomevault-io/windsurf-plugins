---
trigger: always_on
description: validateStatus: () => true,
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

Chat2API Manager is an Electron desktop application that provides an OpenAI-compatible API proxy for multiple AI service providers (DeepSeek, GLM, Kimi, MiniMax, Qwen, Z.ai, Perplexity). It enables using any OpenAI-compatible client with these providers across macOS, Windows, and Linux.

## Build Commands

```bash
# Development
npm run dev              # Start dev server (macOS/Linux)
npm run dev:win          # Start dev server (Windows)

# Build
npm run build            # Build the application
npm run build:mac        # Build for macOS (dmg, zip)
npm run build:win        # Build for Windows (nsis)
npm run build:linux      # Build for Linux (AppImage, deb)
npm run build:all        # Build for all platforms

# Preview production build
npm run preview
```

## Architecture

```
src/
├── main/                    # Electron main process
│   ├── index.ts            # App entry point
│   ├── ipc/                # IPC handlers (main ↔ renderer communication)
│   ├── proxy/              # Proxy server (Koa)
│   │   ├── server.ts       # HTTP server with middleware
│   │   ├── forwarder.ts    # Request forwarding logic & auth
│   │   ├── adapters/       # Provider-specific adapters
│   │   ├── routes.ts       # Proxy routes registration
│   │   ├── sessionManager.ts # Multi-turn conversation management
│   │   └── services/       # Prompt injection & prompt generation
│   ├── oauth/              # OAuth authentication
│   │   ├── manager.ts      # OAuth flow orchestration
│   │   ├── inAppLogin.ts   # In-app browser login with token auto-extraction
│   │   └── adapters/       # Provider-specific OAuth adapters
│   ├── providers/          # Provider configurations
│   │   ├── builtin/        # Built-in provider configs (one file per provider)
│   │   └── custom.ts       # Custom provider support
│   ├── store/              # Persistent storage (electron-store)
│   │   ├── store.ts        # Main store manager with IPC bridge
│   │   ├── types.ts        # Type definitions and default values
│   │   └── config.ts       # Configuration management
│   └── tray/               # System tray integration
├── preload/                # Context bridge (IPC API exposure)
├── renderer/               # React frontend
│   ├── components/         # UI components
│   ├── pages/              # Page components
│   ├── stores/             # Zustand state management
│   └── i18n/               # Internationalization (en-US, zh-CN)
└── shared/                 # Shared types between main and renderer
```

## Key Concepts

### Provider Adapters
Each AI provider has a dedicated adapter in `src/main/proxy/adapters/` that handles:
- Message format conversion (OpenAI format → provider-specific format)
- Authentication header construction
- Stream response parsing
- Multi-turn conversation context

To add a new provider:
1. Create config in `src/main/providers/builtin/<provider>.ts`
2. Create OAuth adapter in `src/main/oauth/adapters/<provider>.ts`
3. Create proxy adapter in `src/main/proxy/adapters/<provider>.ts`
4. Create stream handler in `src/main/proxy/adapters/<provider>-stream.ts`
5. Register in `src/main/providers/builtin/index.ts` and `src/main/proxy/adapters/index.ts`

### IPC Communication
All main-renderer communication uses IPC channels defined in `src/main/ipc/channels.ts`. The naming convention is `domain:action` (e.g., `proxy:start`, `accounts:add`).

### Session Management
Multi-turn conversations are managed by `sessionManager.ts`:
- `single` mode: Session deleted after each chat
- `multi` mode: Session persists with parent message IDs for context

### Tool Prompt Injection
For models without native function calling, prompts are injected via `promptInjectionService.ts`. This enables function calling compatibility with clients like Cherry Studio and Kilo Code.

### Session Management Flow
1. Client sends request with `sessionId`
2. `sessionManager.ts` retrieves session or creates new one
3. For `multi` mode: parentMessageId is used to fetch conversation history
4. Adapter creates/uses provider-specific session
5. Response is returned with new parentMessageId for context continuation

## Data Storage

Application data is stored in `~/.chat2api/`:
- `config.json` - Application configuration
- `providers.json` - Provider settings
- `accounts.json` - Account credentials (encrypted)
- `logs/` - Request logs

## Tech Stack

| Component | Technology |
|-----------|------------|
| Framework | Electron 33+ |
| Frontend | React 18 + TypeScript |
| Styling | Tailwind CSS |
| State | Zustand |
| Build | Vite + electron-vite |
| Server | Koa |

## Coding Guidelines

### Immutability (CRITICAL)
ALWAYS create new objects, NEVER mutate existing ones. Use `update` functions that return new copies.

### Error Handling
Handle errors comprehensively:
- Validate all user input before processing
- Provide user-friendly error messages in UI-facing code
- Log detailed error context on the server side
- Never silently swallow errors

### Input Validation
Validate at system boundaries (user input, external APIs). Use schema-based validation where available.

### Security
- Validate all API keys before use
- Sanitize all user inputs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ansujuner/WebChat2api](https://github.com/ansujuner/WebChat2api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
