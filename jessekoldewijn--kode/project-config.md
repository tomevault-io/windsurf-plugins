---
trigger: always_on
description: This document provides AI coding assistants with comprehensive context about the Kode project.
---

# AGENTS.md

This document provides AI coding assistants with comprehensive context about the Kode project.

## Project Overview

**Kode** is a performant, open-source desktop code editor with integrated AI chat support for CLI code agents (similar to Cursor). Built with Tauri v2 (Rust backend) and a custom reactive UI framework called Ripple.

### Key Features

- Modern code editing with Monaco Editor (VS Code's editor, full IntelliSense for JS/TS)
- Syntax highlighting for JS/TS, JSON, Markdown, CSS, HTML, Rust, Python, YAML, and more
- Integrated AI chat panel for interacting with CLI code agents (like OpenCode)
- File system navigation with sidebar file tree
- Integrated terminal emulator (xterm.js with WebGL rendering)
- Command palette for quick actions
- Theme support (dark/light/system)

## Tech Stack

### Frontend

| Technology                    | Version | Purpose                                                     |
| ----------------------------- | ------- | ----------------------------------------------------------- |
| Ripple                        | latest  | TypeScript-first reactive UI framework (`.ripple` files)    |
| TailwindCSS                   | v4      | CSS-based utility styling                                   |
| Vite (rolldown-vite)          | latest  | Build tool                                                  |
| Monaco Editor                 | latest  | Code editor (VS Code's editor engine)                       |
| vite-plugin-monaco-editor-esm | 2.0.2   | Monaco Editor Vite integration (ESM, Node.js 25 compatible) |
| xterm.js                      | v5      | Terminal emulator                                           |
| TypeScript                    | ^5.7    | Language                                                    |

### Backend (Tauri/Rust)

| Technology    | Version                | Purpose                       |
| ------------- | ---------------------- | ----------------------------- |
| Tauri         | v2                     | Desktop application framework |
| Rust          | 2021 edition (1.77.2+) | Backend language              |
| portable-pty  | 0.8                    | Terminal/PTY emulation        |
| tokio         | 1                      | Async runtime                 |
| fuzzy-matcher | 0.3                    | File search                   |
| ignore        | 0.4                    | .gitignore-aware file walking |

### Package Management

- **Node.js:** 25.6.0 (managed via Volta)
- **Yarn:** 4.12.0 (Berry)

## Directory Structure

```
Kode/
├── src/                        # Frontend source code
│   ├── components/             # UI components (.ripple files)
│   │   ├── chat/              # AI chat panel (ChatPanel, ChatInput, ChatMessage)
│   │   ├── editor/            # Code editor (CodeEditor, EditorTabs)
│   │   ├── filetree/          # File explorer (FileTree, FileNode)
│   │   ├── layout/            # Layout components (Sidebar, EditorArea, Panel, StatusBar)
│   │   ├── palette/           # Command palette
│   │   └── terminal/          # Terminal component
│   ├── lib/                   # Utility libraries
│   │   ├── mocks/             # Browser mock system for dev without Tauri
│   │   ├── tauri.ts           # Tauri API wrapper functions
│   │   ├── workspace.ts       # Workspace/file state management
│   │   ├── theme.ts           # Theme management
│   │   └── monaco-theme.ts    # Monaco Editor theme definitions
│   ├── styles/                # Global styles (global.css with Tailwind)
│   ├── App.ripple             # Main application component
│   └── main.ts                # Application entry point
├── src-tauri/                  # Tauri/Rust backend
│   ├── src/
│   │   ├── commands/          # IPC command handlers
│   │   │   ├── filesystem.rs  # File operations (read, write, search)
│   │   │   ├── terminal.rs    # PTY terminal management
│   │   │   └── agent.rs       # AI agent process spawning
│   │   ├── lib.rs             # Main Tauri setup and command registration
│   │   └── main.rs            # Entry point
│   ├── capabilities/          # Tauri security capabilities
│   └── tauri.conf.json        # Tauri configuration
├── tests/                      # Test files
│   ├── components/            # Component tests
│   ├── e2e/                   # Playwright E2E tests
│   ├── lib/                   # Library tests
│   └── setup.ts               # Vitest setup (mocks Tauri APIs)
├── .agents/                    # AI agent skill definitions
│   └── skills/                # Skill files for AI assistants
└── package.json
```

## Development Commands

| Command            | Description                             |
| ------------------ | --------------------------------------- |
| `yarn dev`         | Start Vite dev server (port 1420)       |
| `yarn tauri:dev`   | Start Tauri development with hot reload |
| `yarn build`       | Build frontend for production           |
| `yarn tauri:build` | Build Tauri desktop application         |
| `yarn format`      | Format code with Prettier               |
| `yarn test`        | Run Vitest in watch mode                |
| `yarn test:run`    | Run Vitest once                         |
| `yarn test:e2e`    | Run Playwright E2E tests                |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JesseKoldewijn/Kode](https://github.com/JesseKoldewijn/Kode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
