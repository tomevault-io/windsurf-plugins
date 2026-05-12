---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**GrapWork** - A cross-platform desktop AI Agent assistant (Electron + Vue 3 + TypeScript) that supports any OpenAI-compatible LLM API. Features:
- **Global Memory**: Persistent knowledge storage for user preferences and custom context with smart keyword matching
- **Assistant System**: Custom AI assistant/system prompt management with default "EddieLab-Agent (ELA)"
- **MCP Support**: Model Context Protocol integration for extensible tool/function calling with STDIO/SSE transports
- **Unified Storage**: Pluggable storage backend system (LocalStorage, FileSystem, HTTP) with type-safe APIs
- **Skills System**: Domain knowledge packages that can be injected into AI context (SKILL.md files with frontmatter)
- **Workspace View**: Built-in file browser and management capabilities
- **Multi-chat Support**: Tab-based chat management with search functionality
- **Image Generator**: Multi-provider image generation (Zhipu GLM-Image, Qwen-Image, Qwen-Image-Edit) with session history

## Development Commands

All commands should be run from the `/frontend` directory:

```bash
# Development: Start Electron with hot reload (Vite dev server + Electron main process)
npm run electron:dev

# Build renderer only (Vue app to dist/)
npm run build:renderer

# Build Electron processes only (main/preload to dist-electron/)
npm run build:electron

# Full production build + packaging (creates installers in release/)
npm run electron:build

# Platform-specific builds
npm run electron:build:mac    # macOS only (dmg, zip)
npm run electron:build:win    # Windows only (nsis, zip)
npm run electron:build:all    # Both macOS and Windows

# Watch Electron build during development
npm run build:electron:watch

# Generate application icons
npm run generate-icons
```

For the optional web server (from `/server` directory):
```bash
cd ../server && npm run dev  # Runs Fastify server on port 8787
```

**Server Configuration:**
- Requires `OPENAI_API_KEY` environment variable
- Default port: 8787 (override with `PORT` env var)
- Endpoints:
  - `GET /api/health` - Health check
  - `POST /api/chat/completions` - Full OpenAI-compatible endpoint
  - `POST /api/chat` - Simple chat endpoint (streaming only)

## Architecture

### Process Structure

**Main Process** (`electron/main.ts`):
- Manages Electron BrowserWindow lifecycle
- Handles IPC communication for config and API requests
- MCP client implementation (JSON-RPC 2.0)
- Simple command executor with whitelist for safe shell commands
- Skills file system operations
- Entry point: `dist-electron/main.cjs`

**Renderer Process** (Vue 3 app):
- SPA with hash-based routing
- Components in `src/components/`
- Entry point: `dist/index.html`

### Key Files and Directories

```
frontend/
├── electron/
│   ├── main.ts           # Electron main process (IPC, MCP client, file operations, skills)
│   └── preload.ts        # Context bridge for renderer→main communication
├── src/
│   ├── components/
│   │   ├── ChatView.vue             # Main chat interface with multi-tab support
│   │   ├── NormalChat.vue           # Standard chat mode component
│   │   ├── ChatTabBar.vue           # Tab bar with search functionality
│   │   ├── WorkspaceView.vue        # Workspace/file browser component
│   │   ├── SettingsView.vue         # Unified settings (8 tabs)
│   │   ├── ImageGeneratorView.vue   # Image generation interface (separate window)
│   │   ├── AssistantView.vue        # Assistant system prompt management
│   │   ├── GlobalMemoryView.vue     # Global memory management UI
│   │   ├── MCPView.vue              # MCP server configuration management
│   │   ├── LoginView.vue            # Authentication entry point
│   │   ├── EnvironmentCheckView.vue # Environment dependency check UI
│   │   ├── ChangelogView.vue        # Update changelog display
│   │   ├── SaveToGlobalMemoryDialog.vue
│   │   ├── GlobalMemoryFormDialog.vue
│   │   ├── ConfirmDialog.vue
│   │   ├── HtmlPreviewDialog.vue
│   │   ├── MermaidDialog.vue
│   │   ├── settings/
│   │   │   ├── LLMConfigPanel.vue       # LLM API configuration form
│   │   │   ├── CodeHighlightThemePanel.vue  # Code theme selection
│   │   │   ├── SkillsPanel.vue          # Skills management UI
│   │   │   └── EnvironmentCheckPanel.vue   # Environment check panel
│   │   └── icons/                   # Icon components (40+)
│   ├── composables/
│   │   ├── useGlobalMemory.ts       # Global knowledge storage
│   │   ├── useMCP.ts                # MCP server management
│   │   ├── useSkills.ts             # Skills management
│   │   └── useImageGenerator.ts     # Image generation state management
│   ├── imageProviders/
│   │   ├── index.ts                 # Provider registry and factory
│   │   ├── types.ts                 # Provider interfaces
│   │   ├── zhipu.ts                 # Zhipu GLM-Image provider
│   │   ├── qwen.ts                  # Qwen-Image provider
│   │   └── qwenImageEdit.ts         # Qwen-Image-Edit provider
│   ├── services/
│   │   ├── StorageService.ts        # Unified storage layer (singleton)
│   │   └── storage/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eddie-292/grapwork](https://github.com/eddie-292/grapwork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
