---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Hibiki (響)** is a desktop audio companion for Discord, built as an Electron app. It plays music, ambience, and sound effects in Discord voice channels from a desktop UI. Originally designed for Dungeons & Dragons (background music, ambient soundscapes, one-shot effects) but works for any server.

**Current state:** The project is mid-migration from a monorepo (with separate `apps/bot` and `apps/web`) to a unified Electron app. The old structure has been deleted (see git status). The new `app/` package is the single application.

## Common Commands

**IMPORTANT:** Always run `nvm use` before executing any pnpm commands to activate the correct Node.js version (24.14.0).

All commands run from the **repo root**:

```bash
# First, activate the correct Node version
nvm use

# Development
pnpm dev                # Build and launch Electron app
pnpm build              # Build backend (tsc) + frontend (Vite) without launching

# Quality checks (ALWAYS run after code changes)
pnpm lint               # ESLint on backend and frontend
pnpm lint:fix           # Auto-fix linting issues
pnpm test               # Run Jest (backend) + Vitest (frontend) tests

# E2E tests
pnpm run build          # Build first
pnpm test:e2e           # Playwright tests against Electron app

# Distribution (Electron Forge handles everything automatically)
pnpm dist               # Package for current platform (alias for pnpm make)
pnpm dist:mac           # macOS (ZIP for x64 and arm64)
pnpm dist:win           # Windows (Squirrel installer + ZIP)
pnpm dist:linux         # Linux (DEB + ZIP)
pnpm make               # Primary build command (same as pnpm dist)
pnpm package            # Create unpacked app only (for testing)
```

### Testing Individual Components

```bash
# Backend tests (Jest)
cd app && pnpm run test:backend           # All backend tests
cd app && pnpm run test:watch             # Watch mode

# Frontend tests (Vitest)
cd app && pnpm run test:frontend          # All frontend tests
cd app/frontend && vitest run <file>      # Single test file

# Coverage
pnpm run test:coverage                    # Both backend + frontend
cd app && pnpm run test:coverage:backend  # Backend only
cd app && pnpm run test:coverage:frontend # Frontend only
```

## Architecture

### Electron Structure

```text
app/
├── electron/          # Electron main process
│   ├── main.js        # Entry point: boots backend, creates window, IPC handlers
│   ├── preload.js     # Bridge: exposes safe IPC to renderer
│   ├── splash.html    # Splash screen shown during app initialization
│   └── splash-logo.png # Logo for splash screen
├── src/               # Backend (runs in Electron main process)
│   ├── bootstrap-embedded.ts    # Wires services, exposes IPC API
│   ├── discord/       # Discord.js client (login, voice, guild directory)
│   ├── player/        # Playback engine (voice connections, audio streams, volume)
│   ├── sound/         # Sound library (list, upload, delete files)
│   ├── scenes/        # Scene store + import/export (soundboards)
│   ├── audio/         # Audio mixing (guild-specific managers)
│   ├── config/        # Configuration (env + validation)
│   └── persistence.ts # JSON key-value store (app-config.json)
├── frontend/          # Vue 3 renderer (Electron renderer process)
│   └── src/
│       ├── api/       # IPC wrappers (typed frontend API calling backend)
│       ├── audio/     # Web Audio capture (AudioWorklet for browser streaming)
│       ├── stores/    # Pinia stores (player state, guild directory)
│       └── views/     # Pages (Welcome, Scenes, Browser, Media, Settings)
├── web-dist/          # Built frontend (Vite output)
└── dist/              # Compiled backend (tsc output)
```

### Communication: IPC Only (No HTTP)

- **Frontend → Backend:** All communication uses Electron IPC via the `preload.js` bridge.
- **IPC API:** Defined in `bootstrap-embedded.ts` as the `EmbeddedApi` interface.
- **Domains:** `player`, `config`, `sounds`, `scenes` (matching the service structure).
- Frontend wrappers in `frontend/src/api/` provide typed functions (e.g., `joinChannel()`, `listScenes()`).
- Audio streaming (Browser feature) uses chunked IPC: `audio:startStream`, `audio:chunk`, `audio:stopStream`.

### Multi-Guild Architecture

- Bot can be in multiple Discord servers simultaneously.
- **One voice channel per guild** — each guild has independent:
  - Voice connection
  - Playback state (playing/stopped, current sound)
  - Volume levels (music, effects, ambience)
- `GuildAudioManager` (`src/audio/guild-audio.manager.ts`) manages per-guild state.
- Player (`src/player/player.ts`) coordinates all guild managers.

### Scenes System

**Scene** = soundboard with three categories:
- **Music:** Tracks with volume, loop option
- **Ambience:** Loops with random interval repeats, volume, enabled/disabled toggle
- **Effects:** One-shot sounds

Stored in `scenes.json`. Import/export bundles scenes with their sound files as `.hibiki.zip` archives.

### Storage

Data lives in platform user data directory (e.g., `~/Library/Application Support/hibiki` on macOS):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PhyberApex/hibiki](https://github.com/PhyberApex/hibiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
