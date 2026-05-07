---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with the PCGamingWiki Game Tweaker client.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with the PCGamingWiki Game Tweaker client.

## Project Overview

This is the **PCGamingWiki Game Tweaker** desktop client - an Electron application that provides a GUI for automating game configuration tweaks. It communicates with a Mastra-based AI agent server via WebSocket to process tweak requests, with local tool execution and user approval workflows.

## Tech Stack

- **Electron** - Desktop application framework
- **SvelteKit** - Frontend framework with Svelte 5
- **TypeScript** - Type safety across all processes
- **Tailwind CSS** - Utility-first CSS framework
- **Vite** - Build tool and dev server

## Development Commands

```bash
npm run dev          # Run development servers (TypeScript checking, Electron, SvelteKit)
npm run build        # Build both frontend and Electron app
npm run build:win    # Build for Windows (NSIS + portable)
npm run lint         # Lint entire codebase
npm run typecheck    # Type check all code (main process, preload, AND renderer)
npm run check        # Run all checks (lint + format:check + typecheck)
```

**Note:** To check for TypeScript errors, use `npm run typecheck`. This runs three separate checks:
- `typecheck:node` - Main process and preload scripts (tsconfig.node.json)
- `typecheck:web` - Web/shared types (tsconfig.web.json)
- `typecheck:renderer` - Renderer/SvelteKit (svelte-check)

## Architecture

### Process Separation

```
┌─────────────────────────────────────────────────────────────┐
│                   Renderer (SvelteKit)                       │
│  Components → Hooks → Stores → window.api (Preload)         │
└─────────────────────────────────┬───────────────────────────┘
                                  │ IPC (ipcRenderer.invoke/send)
                                  ▼
┌─────────────────────────────────────────────────────────────┐
│                    Preload (contextBridge)                   │
│  Secure API Bridge - exposes main process functions          │
└─────────────────────────────────┬───────────────────────────┘
                                  │ ipcMain.handle/on
                                  ▼
┌─────────────────────────────────────────────────────────────┐
│                  Main Process (Node.js)                      │
│  IPC Handlers → Services → Agent/Tools/Utils                │
└─────────────────────────────────┬───────────────────────────┘
                                  │ WebSocket/HTTP
                                  ▼
                         Remote Server/Agent
```

### Directory Structure

```
src/
├── main/              # Electron main process
│   ├── services/      # Business logic layer
│   ├── tools/         # Tool implementations (file I/O, game launchers, graphics mods)
│   ├── interfaces/    # Type definitions
│   ├── schemas/       # Zod validation schemas
│   └── index.ts       # Main entry point
├── preload/           # IPC bridge (secure API exposure)
│   └── index.ts       # contextBridge API definitions
└── renderer/          # SvelteKit frontend
    └── src/
        ├── lib/
        │   ├── components/  # Svelte UI components
        │   ├── hooks/       # Custom Svelte hooks
        │   ├── stores/      # Reactive state management
        │   └── utils/       # Helper functions
        └── routes/          # SvelteKit page routes
```

## Main Process Services (`src/main/services/`)

### Core Services

| Service | Purpose |
|---------|---------|
| **AgentService** | WebSocket communication with remote server. Handles agent status, tweak processing, and message routing. |
| **ToolExecutorService** | Executes tools locally. Delegates to tool registry, handles approval flow. |
| **ToolStatusService** | Tracks tool approval/execution status. Manages approval polling and auto-approval of read-only tools. |
| **GameLibraryService** | Aggregates games from launcher services (Steam, Epic, GOG). Singleton with poster update notifications. |
| **SettingsService** | Persistent settings management with listeners for reactive updates. |
| **AppliedTweaksService** | Persists applied tweaks for the revert system. Manages JSON data in app data directory. |
| **RevertService** | Executes revert operations on completed tweaks. Orchestrates rollback and cleanup. |
| **RecipeService** | Handles recipe lookup and replay execution. Compares semver versions and resolves tool arguments. |
| **PCGamingWikiService** | Fetches game data from PCGamingWiki via server proxy. Expands local paths in config files. |
| **TweakMetadataService** | Batch fetches tweak metadata (processability status and recipes) from server API. |
| **SystemSpecsService** | Collects system information (CPU, GPU, memory, OS, display). |
| **UpdaterService** | Auto-update functionality using electron-updater. |
| **AgentAvailabilityService** | Polls server health endpoint (30s intervals). Tracks agent availability. |

### IPC Handlers (`src/main/services/ipc-handlers/`)

| Handler | Key Channels |
|---------|-------------|
| **agent-ipc.handler.ts** | `agent:process-tweak`, `agent:approve-tool`, `agent:decline-tool`, `agent:abort-task` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elisoftli/twiki](https://github.com/elisoftli/twiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
