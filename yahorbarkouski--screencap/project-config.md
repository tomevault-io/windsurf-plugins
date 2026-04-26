---
trigger: always_on
description: Guidance for Codex when working with this repository.
---

# AGENTS.md

Guidance for Codex when working with this repository.

## Project Overview

**Screencap** is a macOS Electron desktop app that captures screenshots on a schedule, classifies them using optional LLM analysis, and turns them into timelines, daily summaries ("Day Wrapped"), and journals. It tracks app/window context, browser activity, media playback, and IDE workspace info to help users understand where their time goes.

Key features:
- Scheduled screenshot capture with context detection (app, window title, browser URL, media, IDE workspace)
- Event timeline with automatic merging of similar captures
- LLM-based classification (local or cloud) with fallback strategies
- Daily journal entries and "Day Wrapped" tray widget
- Project tracking with Git integration
- Addiction tracking with confirmation/rejection workflow
- Social features: sharing projects, friend feeds, encrypted room collaboration
- Privacy-focused: local-first with optional cloud AI (text-first, vision fallback)

## Commands

### Development
```bash
npm install         # Install dependencies
npm run dev         # Start dev server (rebuilds native modules)
npm run dev:fast    # Start dev server (skip native rebuild)
npm test            # Run tests (rebuilds for Node first)
npm run test:watch  # Run tests in watch mode
```

### Building
```bash
npm run build    # Build for production
npm run preview  # Preview production build
npx electron-builder --config electron-builder.yml  # Package distributable
```

### Code Quality
```bash
npm run lint       # Check code with Biome
npm run lint:fix   # Auto-fix linting issues
npm run typecheck  # TypeScript type checking
```

### Utilities
```bash
npm run rebuild:electron  # Rebuild native modules for Electron
npm run rebuild:node      # Rebuild native modules for Node (tests)
npm run generate:icon     # Generate app icon from source
npm run eval:classification -- --limit=25 --strategies=vision,text,local
```

### Single Test Execution
```bash
npm test -- electron/main/features/context/__tests__/resolvers.test.ts
npm test -- -t "spotifyResolver"
```

## Architecture

### Directory Structure

```
electron/
  main/           # Main process (Node.js)
    app/          # App lifecycle (window, tray, popup, protocol, bootstrap)
    features/     # Business logic organized by domain
    infra/        # Infrastructure (db, settings, logging, paths)
    ipc/          # IPC handlers (security boundary)
    testUtils/    # Test utilities
  preload/        # Preload script (contextBridge)
  shared/         # Types and channel names shared between main/renderer
src/              # Renderer process (React UI)
  components/     # UI components organized by feature
  hooks/          # React hooks
  lib/            # Utilities
  stores/         # Zustand state stores
```

### Main Process Features (electron/main/features/)

**Core:**
- `capture/` - Screenshot capture logic (dominant activity sampling)
- `scheduler/` - Interval-based capture scheduling
- `queue/` - Async task processing (classification, OCR, merging)
- `events/` - Event storage, querying, and mutations
- `context/` - Context detection (apps, browsers, media, IDE)
- `ai/` - AI routing and provider abstraction
- `llm/` - LLM API clients (OpenRouter, local endpoints), ClassificationService
- `ocr/` - macOS Vision-based OCR
- `retention/` - Storage cleanup policies

**Supporting:**
- `activityWindow/` - Activity window management
- `automationRules/` - Per-app/website relabeling and dismissal rules
- `projectJournal/` - Git repo linking, commit history
- `projects/` - Project management
- `favicons/` - Safe favicon fetching
- `permissions/` - macOS permission status
- `shortcuts/` - Global hotkeys
- `update/` - Auto-updater (electron-updater)
- `appIcons/` - App icon extraction
- `aiEval/` - Classification evaluation harness

**Social:**
- `social/` - Username registration, friend requests, avatar sync
- `publishing/` - Project share creation with E2EE
- `rooms/` - Project collaboration rooms
- `sharedProjects/` - Syncing shared project events
- `socialFeed/` - Friends feed, day wrapped snapshots
- `chat/` - Direct messages and project threads
- `sync/` - Data synchronization

### Infrastructure (electron/main/infra/)

- `db/` - SQLite schema, migrations, repositories (better-sqlite3)
- `settings/` - Settings storage with encrypted API keys (safeStorage)
- `log/` - File-based logging with rotation
- `paths/` - Path utilities (screenshots, database, user data)
- `windows/` - Window management utilities

### Renderer Components (src/components/)

- `timeline/` - Event timeline view
- `wrapped/` - Day Wrapped visualization
- `settings/` - Settings panels
- `popup/` - Tray popup UI
- `eod/` - End-of-day journal flow
- `progress/` - Project progress timeline
- `memory/` - Memories, projects, addictions views
- `story/` - Story/recap views
- `onboarding/` - Onboarding wizard
- `layout/` - Layout components (titlebar, navigation)
- `visualization/` - Charts and visualizations
- `avatar/` - User avatar components
- `preview/` - Event preview modal
- `dialogs/` - Dialog components
- `performance/` - Performance guards
- `ui/` - Reusable UI components (shadcn/ui-style)

### IPC Security

All IPC goes through a hardened security layer:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yahorbarkouski/screencap](https://github.com/yahorbarkouski/screencap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
