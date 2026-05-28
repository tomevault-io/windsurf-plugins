---
trigger: always_on
description: This file provides guidance to Gemini CLI when working with code in this repository.
---

# GEMINI.md

This file provides guidance to Gemini CLI when working with code in this repository.

## Essential Commands

**Development:**

- `npm run package` - **(Standard)** Build a local production application (in `out/`). This is the recommended way to test changes.
- `npm start` or `npm run dev` - **(Deprecated for testing)** Start development mode with hot reload. Do not use.
- `npm run make` - Unsupported. Do not use.

**Testing & Quality:**

- `npm test` - Run Jest unit tests (includes both main and renderer process tests)
- `npm run test:watch` - Run tests in watch mode
- `npm run lint` - **Note: Currently non-functional** due to ESLint configuration incompatibility (`.eslintrc.js` uses old format incompatible with ESLint 9+). Prettier is used for code formatting.

**Platform-specific builds:**

- `npm run build:win` - Build for Windows
- `npm run build:linux` - Build for Linux

## Architecture Overview

Athanor is an Electron desktop application for AI-assisted development workflows. It helps developers create context-rich prompts and apply AI-generated changes to codebases.

### Core Architecture Principles

**Main/Renderer Separation:**

- **Main Process** (`electron/`): File system operations, Git integration, project analysis
- **Renderer Process** (`src/`): React UI, state management with Zustand
- **IPC Communication**: Secure bridge via `preload.ts` and typed in `src/types/global.d.ts`

**Key Services:**

- `FileService.ts` - Central file system operations manager (main process)
- `RelevanceEngineService.ts` - Intelligent context discovery using multiple heuristics
- `ProjectGraphService.ts` - Background project analysis and dependency mapping
- `GitService.ts` - Git repository analysis for relevance scoring
- `UserActivityService.ts` - Real-time file activity tracking for relevance signals
- `SettingsService.ts` - Project and application settings management
- `ShellService.ts` - Terminal/CLI session management with persistent PTY sessions
- `TaskAnalysisUtils.ts` - Task description analysis and keyword extraction
- `DependencyResolver.ts` / `DependencyScanner.ts` - Language-aware dependency analysis

### Critical File System Patterns

**Always use proper abstraction layers:**

- Main process: Use `FileService.ts` singleton for all file operations
- Renderer process: Use `fileSystemService.ts` for UI-related file operations
- Never bypass IPC for file operations from renderer

**Global type definitions:**

- `src/types/global.d.ts` - Extends window interface and defines core types
- Must be updated when modifying IPC method signatures

### State Management

**Zustand stores in `src/stores/`:**

- `fileSystemStore.ts` - Selected files, file tree, preview state
- `workbenchStore.ts` - Multi-tab task management
- `contextStore.ts` - Intelligent context builder state
- `promptStore.ts` - Prompt templates and variants
- `taskStore.ts` - Task templates and variants
- `applyChangesStore.ts` - AI-generated file change management
- `settingsStore.ts` - Project and application settings state
- `logStore.ts` - Application log management with interactive entries
- `commandStore.ts` - Clipboard and command validation state
- `cliStore.ts` - Terminal session state management

### Project Structure Insights

**Core Intelligence:**

- Relevance Engine uses Git history, dependencies, file mentions, and user activity
- Project analysis runs in background worker thread (`projectAnalysisWorker.ts`)
- Results cached in `.ath_materials/project_graph.json`
- Two-phase scoring engine with multiple heuristics for relevance
- Automatic re-analysis when file changes are detected after inactivity

**AI Integration:**

- Optional direct API integration via secure storage (`electron/modules/secure-api-storage/`)
- Primary workflow: copy prompts to external AI, paste responses back
- XML command parsing for applying AI-generated changes
- Modular LLM provider system (`electron/modules/llm/`) supporting:
  - Anthropic Claude API
  - OpenAI GPT models
  - Google Gemini models
  - Mistral models (API key storage only)
- Type-safe IPC channels for LLM operations
- Extensive model configuration and client adapters

**File Management:**

- Supports `.athignore` and `.gitignore` patterns
- Chokidar file watchers for real-time updates
- Path normalization via `PathUtils.ts`
- Agent task creation in `.ath_materials` directory

**CLI/Terminal Support:**

- Integrated terminal via `node-pty` and `xterm.js`
- Multi-session support with persistent terminals
- Platform-specific shell detection (PowerShell on Windows, zsh/bash on Unix)
- Managed by `ShellService.ts` and `cliStore.ts`

**Additional Features:**

- **Tooltips**: Contextual help throughout the UI via hover tooltips
- **Drag and Drop**: File paths can be dragged from explorer to task/context areas
- **Context Suggestions**: Automatic context suggestions based on task content
- **Preset Tasks**: Pre-defined task templates loaded from `task_*.xml` files
- **Token Budgeting**: Intelligent file inclusion based on token limits
- **Smart Preview**: Configurable preview of file contents in prompts
- **Documentation Format**: Multiple format options for file inclusion
- **Ignore Rules**: Advanced pattern matching with `.athignore` and `.gitignore`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lacerbi/athanor](https://github.com/lacerbi/athanor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
