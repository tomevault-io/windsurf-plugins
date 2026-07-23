---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**NeuroOS** is an AI-powered desktop operating system built with Electron, React, and TypeScript. It provides a windowed desktop environment with an integrated AI assistant that can execute tools, manage files, and control applications. The system features multi-user authentication, workspace management, and integration with multiple LLM providers (Gemini, OpenAI, Ollama) and external services via Composio.

## Development Commands

### Setup & Installation
```bash
npm install                    # Install dependencies
npm run lint                   # TypeScript type checking (no emit)
```

### Development
```bash
npm run dev                    # Start Vite dev server only (web mode, port 5173)
npm run electron:dev           # Start full Electron + Vite dev environment (recommended)
npm run preview               # Preview production build locally
```

### Building
```bash
npm run build                 # Full build: TypeScript + Vite + Electron
npm run electron:build        # Build production Electron app with installer
npm run clean                 # Remove build artifacts (dist, dist-electron)
```

### Key Notes
- **Development**: Use `npm run electron:dev` for full desktop app development. This runs Vite on port 5173 and launches Electron with hot reload.
- **Type Checking**: Run `npm run lint` before commits to catch TypeScript errors.
- **Build Output**: Production builds output to `release/{version}/` with NSIS installer for Windows.

## Architecture Overview

### High-Level Structure

NeuroOS follows a **layered architecture** with clear separation between the Electron main process, React frontend, and AI/tool systems:

```
┌─────────────────────────────────────────────────────────────┐
│                    React Frontend (src/)                     │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ App.tsx (root) → Auth/Onboarding → Desktop Shell    │   │
│  │  ├─ WindowManager (app lifecycle)                   │   │
│  │  ├─ Taskbar, StartMenu, Desktop                     │   │
│  │  └─ Apps (Chat, FileExplorer, Settings, etc.)       │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ State Management (Zustand stores)                    │   │
│  │  ├─ authStore (users, auth state)                   │   │
│  │  ├─ settingsStore (AI config, theme)                │   │
│  │  ├─ workspaceStore (file system path)               │   │
│  │  ├─ sessionStore (chat history)                     │   │
│  │  └─ composioStore (external integrations)           │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ AI & Tool Systems (src/lib/ai/)                      │   │
│  │  ├─ crew.ts (multi-agent orchestration)             │   │
│  │  ├─ toolEngine.ts (tool registry & execution)       │   │
│  │  ├─ tools/ (business, composio, OS tools)           │   │
│  │  └─ llm/factory.ts (provider abstraction)           │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ↕ IPC
┌─────────────────────────────────────────────────────────────┐
│              Electron Main Process (native-shell/)           │
│  ├─ main.ts (window creation, IPC handlers)                 │
│  ├─ preload.ts (context bridge for secure API)              │
│  └─ File system, app lifecycle, system integration          │
└─────────────────────────────────────────────────────────────┘
```

### Key Architectural Patterns

#### 1. **IPC Communication (Electron ↔ React)**
- **Frontend → Main**: React calls `window.electron.*` methods (exposed via preload.ts)
- **Main → Frontend**: IPC events trigger React state updates
- **File System**: All file operations go through IPC to the main process for security
- **Path Validation**: `isPathSafe()` in main.ts validates all user-provided paths

#### 2. **State Management (Zustand)**
All state is managed via Zustand stores with persistence middleware:
- **authStore**: User profiles, authentication, PIN verification (SHA-256 hashed)
- **settingsStore**: AI provider config, theme, user preferences
- **workspaceStore**: Selected workspace path (persisted across restarts)
- **sessionStore**: Chat history and messages
- **composioStore**: External service integrations and permissions

#### 3. **AI & Tool Execution**
- **Tool Registry** (`toolEngine.ts`): Central registry of all available tools
- **Tool Categories**: `os`, `file`, `shell`, `browser`, `generate`, `automation`, `business`
- **Tool Execution**: `executeTool()` runs tools with context (file access, app control, etc.)
- **Multi-Agent System** (`crew.ts`): Agents (Coordinator, Planner, Researcher, etc.) with specialized roles and tool access
- **Composio Integration**: External app integrations (email, Slack, GitHub, etc.) via Composio SDK


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [loayabdalslam/NeuroOS](https://github.com/loayabdalslam/NeuroOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
