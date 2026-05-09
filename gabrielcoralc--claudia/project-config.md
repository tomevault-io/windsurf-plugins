---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claudia is an Electron-based macOS desktop application for managing Claude Code sessions. It provides a visual interface for tracking multiple sessions, viewing chat history, monitoring costs, and managing terminal instances.

## Essential Commands

### Development
```bash
npm run dev              # Start in development mode with hot reload
npm run build            # Build the app for production
npm run preview          # Preview the production build
```

### Testing & Quality
```bash
npm run test             # Run all tests once
npm run test:watch       # Run tests in watch mode
npm run lint             # Run ESLint
npm run typecheck        # Run TypeScript type checking
npm run security:check   # Run security audit + lint + typecheck
```

### Building for Distribution
```bash
npm run package:mac      # Build .dmg for macOS (arm64 by default)
npm run postinstall      # Rebuild native modules (better-sqlite3, node-pty)
```

**Important**: After adding or updating native dependencies (better-sqlite3, node-pty), always run `npm run postinstall` to rebuild them for Electron's runtime.

## Architecture

### Process Model (Electron)

- **Main Process** (`src/main/`): Node.js backend with full system access
  - Entry point: `src/main/index.ts`
  - Manages window lifecycle, services, and IPC handlers

- **Renderer Process** (`src/renderer/`): React frontend with restricted access
  - Entry point: `src/renderer/src/main.tsx`
  - Communicates with main process via IPC through preload bridge

- **Preload Script** (`src/preload/`): Secure IPC bridge
  - Exposes `window.api` to renderer with type-safe methods
  - Context isolation enabled for security

### Service Layer Architecture

All core functionality lives in `src/main/services/`:

#### Database Service (`Database.ts`)
- SQLite database using better-sqlite3 with WAL mode
- Schema: sessions, messages, projects, settings, reviews, session_daily_metrics
- Foreign keys enabled for referential integrity
- Exported interfaces: `sessionDb`, `messageDb`, `projectDb`, `settingsDb`, `reviewDb`, `analyticsDb`, `dailyMetricsDb`

#### Terminal Service (`TerminalService.ts`)
- Manages multiple PTY (pseudo-terminal) instances using node-pty
- One terminal per session, tracked by session ID
- Supports git operations: diff, revert, stash, branch detection
- Terminal lifecycle: create → write → resize → kill

#### SessionParser Service (`SessionParser.ts`)
- Parses Claude Code transcript files (line-delimited JSON)
- Decodes project paths from Claude's encoding scheme (`/Users/foo/bar` → `-Users-foo-bar`)
- Extracts session metadata: title, costs, token usage, messages
- Transforms `AskUserQuestion` tool calls into readable markdown
- Incremental parsing support for large transcripts

#### FileWatcher Service (`FileWatcher.ts`)
- Watches `~/.claude/projects/` for transcript changes using chokidar
- Detects new sessions and updates automatically
- Handles both app-launched and externally-started sessions
- Debounced file change detection to avoid duplicate processing

#### PricingService (`PricingService.ts`)
- Calculates session costs based on token usage
- Fetches latest pricing from Anthropic's website on startup (non-blocking)
- Falls back to bundled pricing.json if fetch fails
- Supports longest-match model ID resolution (e.g., `claude-sonnet-4-5-20250929` → `claude-sonnet-4-5`)
- Cache token pricing included (read and write)

#### WindowManager Service (`WindowManager.ts`)
- Centralized window reference management
- Provides `sendToRenderer()` for push notifications to UI
- Used by FileWatcher and other services to update UI in real-time

#### HooksServer Service (`HooksServer.ts`)
- Local HTTP server for Claude Code hooks integration
- Receives callbacks when Claude starts/resumes sessions
- Optional feature (disabled by default)

#### AutoUpdater Service (`AutoUpdater.ts`)
- Checks GitHub releases for new versions
- Background downloads with progress tracking
- One-click install workflow

### State Management

Zustand store (`src/renderer/src/stores/sessionStore.ts`):
- Sessions list and selected session
- Messages cache (lazy-loaded per session)
- Active terminals tracking
- Subsession parent-child relationships
- Settings state

### IPC Communication Pattern

All renderer → main communication goes through IPC handlers in `src/main/ipc/handlers.ts`:

```typescript
// Renderer side (React components)
const sessions = await window.api.sessions.list()

// Main process (handlers.ts)
ipcMain.handle('sessions:list', () => sessionDb.getAll())
```

Key IPC namespaces:
- `sessions:*` - Session CRUD, messages, status updates
- `projects:*` - Project listing, git operations
- `settings:*` - App configuration
- `terminal:*` - Terminal lifecycle, input/output
- `analytics:*` - Cost and usage metrics
- `reviews:*` - Session review storage

### Claude Code Integration

**Session Discovery**:
- Claude Code stores sessions in `~/.claude/projects/<encoded-project-path>/sessions/`
- Each session has a transcript file: `<session-id>.jsonl`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabrielcoralc/claudia](https://github.com/gabrielcoralc/claudia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
