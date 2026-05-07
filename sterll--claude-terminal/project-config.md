---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Terminal is a cross-platform Electron desktop application (**v1.2.6**) for managing Claude Code projects. It bundles an integrated terminal, a Claude Agent SDK chat UI, full Git workflow, a visual workflow automation editor (LiteGraph), parallel task orchestration, a Control Tower for multi-agent supervision, a workspace knowledge base, cloud sync, a PWA remote control, and a plugin/skill ecosystem. Primary target: Windows 10/11 with NSIS installer. Also builds for macOS (DMG) and Linux (AppImage / Snap / Flatpak).

**Repository:** `github.com/Sterll/claude-terminal` | **License:** GPL-3.0 | **Author:** Yanis

## Build & Development Commands

```bash
npm install              # Install dependencies (Node >=18, runs electron-rebuild for node-pty, keytar, better-sqlite3)
npm start                # Build renderer + run app
npm run start:dev        # Run with DevTools enabled
npm run start:inspect    # Run with remote debugging port 9222
npm run watch            # Build renderer in watch mode (esbuild)
npm run build:renderer   # Build renderer only -> dist/renderer.bundle.js
npm run build            # Build installer for current platform -> build/
npm run build:win        # Windows NSIS installer
npm run build:mac        # macOS DMG
npm run build:linux      # Linux AppImage
npm run publish          # Build and publish Windows installer to update server
npm test                 # Run Jest tests (jsdom, 40 test files)
npm run test:watch       # Jest in watch mode
```

**Important:** Always run `npm run build:renderer` after modifying anything under `src/renderer/`, `src/project-types/`, or `renderer.js`.

## Architecture Overview

```
Electron Main Process (Node.js)
├── main.js                          # Bootstrap, lifecycle, single-instance lock, global shortcuts
├── src/main/preload.js              # IPC bridge (window.electron_api)
├── src/main/preload-quickpicker.js  # Preload for Quick Picker window
├── src/main/ipc/                    # 26 IPC files, 256 handlers total
├── src/main/services/               # 24 services
├── src/main/windows/                # 5 window managers
├── src/main/utils/                  # 9 utilities
├── src/main/workflow-nodes/         # 21 workflow node types
└── src/main/workflow-triggers/      # 6 trigger types

Electron Renderer Process (Browser)
├── renderer.js                      # Entry point (bundled by esbuild -> dist/renderer.bundle.js)
├── src/renderer/index.js            # Module loader & initialization
├── src/renderer/core/               # DI container, BaseService/Component/Panel, ApiProvider
├── src/renderer/state/              # 12 observable state modules
├── src/renderer/services/           # 19 services + modular markdown renderer
├── src/renderer/ui/components/      # 11 UI components
├── src/renderer/ui/panels/          # 20 UI panels
├── src/renderer/features/           # Keyboard shortcuts, quick picker, drag-drop
├── src/renderer/events/             # Claude event bus (hook + scraping providers)
├── src/renderer/workflow-fields/    # 14 custom UI fields for workflow nodes
├── src/renderer/workflow-triggers/  # Renderer-side trigger configurators
├── src/renderer/viewers/            # PDF viewer + 3D (three.js) viewer
├── src/renderer/i18n/               # EN/FR/ES locales (~1400 keys each)
└── src/renderer/utils/              # DOM, color, format, paths, icons, syntax highlighting

Project Types (Plugin System)
└── src/project-types/               # general, api, fivem, minecraft, python, webapp, discord

Shared code
└── src/shared/                      # workflow-schema.js shared between main and renderer

Styles
└── styles/                          # 26 modular CSS files (~50,700 lines total)

MCP Servers (shipped with the app)
└── resources/mcp-servers/
    ├── claude-terminal-mcp.js       # Unified MCP server
    ├── database-mcp-server.js       # Specialized DB server
    └── tools/                       # 19 tool modules (auto-registered)

Remote UI (PWA for mobile)
└── remote-ui/                       # Web interface bundled as extraResources
```

## Main Process (`src/main/`)

### IPC Handlers (`src/main/ipc/`)

| File | Handlers | Key Operations |
|------|---------:|----------------|
| `terminal.ipc.js` | 4 | Create PTY (node-pty), input, resize, kill |
| `git.ipc.js` | 69 | Status, branches, pull/push/merge/rebase, clone, stash, cherry-pick, revert, tag, blame, worktree, AI commit message, PR description, inline diff |
| `github.ipc.js` | 26 | OAuth Device Flow, workflow runs, PRs, issues, reviews, GitHub Enterprise, repo search |
| `chat.ipc.js` | 16 | Agent SDK streaming sessions, permissions, interrupt, model/effort switching, tab name generation, fork/rewind, skill/agent generation, session recap |
| `dialog.ipc.js` | 21 | Window controls, file/folder dialogs, open in explorer/editor/browser, notifications, updates, startup, clipboard |
| `explorer.ipc.js` | 3 | File explorer watcher (start/stop/onChanges) |
| `mcp.ipc.js` | 2 | Start/stop MCP server processes |
| `mcpRegistry.ipc.js` | 3 | Browse/search/detail `registry.modelcontextprotocol.io` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sterll/claude-terminal](https://github.com/Sterll/claude-terminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
