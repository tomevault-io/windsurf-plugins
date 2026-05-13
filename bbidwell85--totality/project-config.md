---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Totality is an Electron desktop application that analyzes media library quality from multiple sources (Plex, Jellyfin, Emby, Kodi, MediaMonkey) and recommends higher-quality versions. Includes tag sync (mood/genre) across sources with bidirectional write support. Built with Electron 41, React 18, TypeScript, Vite 6, and Tailwind CSS 4.

## Development Commands

```bash
npm run electron:dev     # Start Vite + Electron together (recommended for development)
npm run build            # TypeScript compile + Vite build + Electron Builder
npm run lint             # Run ESLint (flat config, eslint.config.js)
npm run preview          # Preview Vite production build
npm run test             # Run Vitest in watch mode
npm run test:run         # Run all tests once
npm run test:coverage    # Run tests with coverage report
npm run test:ui          # Open Vitest interactive UI in browser
npm run generate-icons   # Generate app icons from source
npm run electron:build   # Same as `npm run build` (alias)
```

**Note:** `postinstall` runs `electron-rebuild` automatically after `npm install` to compile native modules (better-sqlite3) for Electron's Node ABI.

**Single test:** `npx vitest run tests/unit/FileNameParser.test.ts` or use `-t` to filter by test name: `npx vitest run -t "parses year"`

DevTools open automatically in development mode (docked to bottom).

**Note:** `npm run dev` and `npm run electron:dev` are equivalent—both run Vite, which automatically starts Electron via `vite-plugin-electron`.

**Build Output:** `dist/` (renderer), `dist-electron/` (main + preload), `release/` (packaged apps)

## Versioning & Releases

Uses [standard-version](https://github.com/conventional-changelog/standard-version) for automatic semantic versioning based on conventional commits.

### Release Commands

```bash
npm run release          # Auto-detect version bump from commits
npm run release:patch    # Force patch bump (0.1.0 → 0.1.1)
npm run release:minor    # Force minor bump (0.1.0 → 0.2.0)
npm run release:major    # Force major bump (0.1.0 → 1.0.0)
npm run release:beta     # Pre-release beta bump (0.1.0 → 0.1.1-beta.0)
```

### How It Works

1. Scans git history for conventional commits since last tag
2. Determines version bump: `fix:` → patch, `feat:` → minor, `BREAKING CHANGE:` → major
3. Updates `package.json` version
4. Generates/updates `CHANGELOG.md`
5. Creates git commit and tag (e.g., `v0.1.1`)

### Commit Message Format

```
type(scope): description

feat: add new provider support        → minor bump
fix: resolve scan timeout issue       → patch bump
feat!: redesign settings API          → major bump (breaking)
```

**Note:** After running a release command, push with tags: `git push --follow-tags`

### Development Database

Location: `%APPDATA%\totality\totality.db` (SQLite via better-sqlite3 in production, SQL.js as fallback)

Reset database: `del "%APPDATA%\totality\totality.db"`

## Architecture Overview

### Three-Process Model (Electron Standard)

**1. Main Process** (`src/main/`)
- Node.js environment with full system access
- Window management, database operations (better-sqlite3/SQL.js), external API communication
- Entry: `src/main/index.ts` → builds to `dist-electron/main/index.cjs`

**2. Preload Script** (`src/preload/`)
- Secure bridge between main and renderer via `contextBridge`
- Exposes `window.electronAPI` with typed IPC methods
- Entry: `src/preload/index.ts` → builds to `dist-electron/preload/index.cjs`

**3. Renderer Process** (`src/renderer/`)
- React 18 + TypeScript web application
- Chromium environment, no Node.js access (security)
- Entry: `src/renderer/src/main.tsx` → builds to `dist/`

**Worker Threads:**
- FFprobe worker (`src/main/workers/ffprobe-worker.ts`) — separate Vite entry point, builds to `dist-electron/main/ffprobe-worker.cjs`

### Core Services (Singletons in Main Process)

Services use singleton pattern via `getXxxService()` getter functions (see Singleton Services pattern below). Key services with non-obvious behavior:

- **DatabaseService** (`getDatabaseService()`): Dual SQLite backend — better-sqlite3 (production) or SQL.js (fallback/tests). See Database Backend Selection.
- **SourceManager** (`getSourceManager()`): Orchestrates all provider lifecycles, scanning, connection testing
- **QualityAnalyzer** (`getQualityAnalyzer()`): Tier-based scoring with codec efficiency multipliers
- **GeminiService** (`getGeminiService()`): Sync constructor (async init causes race condition). See AI Chat & Analysis.
- **LoggingService** (`getLoggingService()`): Intercepts console globally, sanitizes sensitive data, verbose mode via `verbose()` method
- **TaskQueueService** (`getTaskQueueService()`): Background task queue — runs quality analysis after music scans

All other services follow the same singleton getter pattern and are discoverable via `grep -r "export function get.*Service"` in `src/main/`.

### Multi-Provider Architecture

**Location:** `src/main/providers/`

The application supports multiple media server providers through a common interface:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bbidwell85/Totality](https://github.com/bbidwell85/Totality) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
