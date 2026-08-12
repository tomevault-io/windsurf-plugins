---
trigger: always_on
description: > Read automatically by Claude Code / AI coding assistants and referenced as the team's engineering conventions.
---

# Millwright — Project Conventions (CLAUDE.md)

> Read automatically by Claude Code / AI coding assistants and referenced as the team's engineering conventions.

## Project Overview

Millwright is an open-source SolidWorks AI automation assistant. Users describe operations in natural language; the AI generates VBA/Python scripts and injects them into SolidWorks through the COM interface. Supports the Anthropic and OpenAI-compatible protocols, so any large model can be plugged in.

- **Tech stack**: Electron 28 + React 18 + TypeScript 5.3 + cscript/VBS (COM) + native fetch/SSE
- **Runtime**: Windows 10/11 (64-bit), SolidWorks 2017+, Node.js 20+
- **Package manager**: npm

## Architecture at a Glance

```
Renderer (React)  ←IPC→  Main Process (Node.js)  ←COM/cscript→  SolidWorks
```

Three-process model:

- **Main** (`src/main/`): LLM calls, script generation/execution, COM bridge, config persistence
- **Preload** (`src/preload/`): contextBridge security boundary exposing `window.api`
- **Renderer** (`src/renderer/`): React UI, pure orchestration layer

## Directory Layout

```
src/
├── shared/                  # Shared between main and renderer (types, constants, presets)
│   ├── types.ts             #   All interfaces & types
│   ├── ipc-channels.ts      #   IPC channel constants (single source of truth)
│   ├── presets.ts           #   Model presets & DEFAULT_CONFIG
│   └── sw-tools.ts          #   26 SW tool definitions (metadata)
├── main/
│   ├── index.ts             #   Application entry, window management
│   ├── ipc/handlers.ts      #   Centralized IPC handler registration
│   ├── llm/                 #   LLM dual-protocol adapters
│   │   ├── adapter.ts       #     BaseLLMAdapter abstract base class
│   │   ├── anthropic.ts     #     Anthropic protocol implementation
│   │   ├── openai.ts        #     OpenAI-compatible protocol implementation
│   │   ├── sse.ts           #     Hand-written SSE streaming parser
│   │   ├── factory.ts       #     createAdapter() factory
│   │   ├── code-extract.ts  #     Code block extraction
│   │   ├── context-window.ts#     Token estimation & message truncation
│   │   ├── errors.ts        #     Error normalization → LLMErrorInfo
│   │   └── prompts.ts       #     System prompts (dynamic context stitching)
│   ├── com/                 #   SolidWorks COM bridge
│   │   ├── sw-bridge.ts     #     cscript/VBS connection management (not winax)
│   │   ├── health.ts        #     Heartbeat monitoring
│   │   ├── context-collector.ts #  Document context collection
│   │   ├── tools.ts         #     Tool metadata export
│   │   └── vbs-writer.ts    #     VBS file writer (UTF-16LE+BOM)
│   ├── scripts/
│   │   ├── engine.ts        #     Script execution engine (cscript > python > com)
│   │   ├── sanitizer.ts     #     Safety check (per-language blacklists for VBA/Python)
│   │   ├── backup.ts        #     Auto-backup before execution
│   │   ├── vba-macro-writer.ts #  VBA → VBS 10-step conversion
│   │   ├── generators/      #     VBA generators for 26 SW tools
│   │   │   ├── index.ts     #       Registry + generateScript() + checkCoverage()
│   │   │   ├── vba-helpers.ts #     mmToM / degToRad / vbaString / wrapMain / selectPlane
│   │   │   ├── sketch.ts    #       Sketches
│   │   │   ├── feature.ts   #       Features (extrude/cut/revolve/fillet/pattern/mirror/dimension)
│   │   │   ├── document.ts  #       Document operations
│   │   │   ├── assembly.ts  #       Assemblies
│   │   │   ├── export.ts    #       Export
│   │   │   └── batch-query.ts #     Batch queries
│   │   └── templates/       #     Prebuilt script templates
│   └── store/
│       ├── config.ts        #     Config persistence (safeStorage-encrypted API key)
│       ├── chat-store.ts    #     Chat history CRUD
│       └── env-fallback.ts  #     .env parsing + protocol mapping
├── preload/
│   └── index.ts             #   contextBridge exposes window.api
└── renderer/
    ├── App.tsx              #   Pure orchestration layer
    ├── components/          #   UI components
    ├── hooks/               #   useLLM / useSWStatus / useTheme
    ├── themes/              #   Light/dark token sets
    └── styles/
```

## Development Commands

```bash
npm install          # Install dependencies
npm run dev          # Dev mode (parallel tsc -w + vite + electron)
npm run build        # Compile main + preload + renderer
npm run dist         # Build NSIS installer
npm test             # Run all tests (node:test, requires build:main first)
npm run lint         # ESLint check
npm run format       # Prettier formatting
```

## Core Development Conventions

### IPC

- **Channel names must only be imported from `shared/ipc-channels.ts`**, never hard-coded as strings.
- The main process uses `ipcMain.handle` (awaitable); streaming events use `webContents.send`.
- The renderer only calls `window.api.xxx()`, never `ipcRenderer` directly.

### LLM Adapters

- New protocols must extend `BaseLLMAdapter` and implement `chat / chatStream / test`.
- Errors are normalized through `toLLMError()`; **never throw raw Error**.
- Streaming is exposed as `AsyncIterable<LLMStreamEvent>`: `start | delta | done | error`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raylanlin/Millwright](https://github.com/raylanlin/Millwright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
