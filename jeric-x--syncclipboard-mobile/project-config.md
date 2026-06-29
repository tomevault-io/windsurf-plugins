---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

```bash
npm install                        # Install dependencies
npm run prebuild                   # Generate native Android/iOS projects
npm run android                    # Build & run on Android device/emulator
npm run ios                        # Run on iOS (if supported)
npm run build:apk                  # Build release APK (android/gradlew assembleRelease)
npm run plugin:build               # Compile Expo config plugins (TypeScript in plugins/)
npm run type-check                 # tsc --noEmit
npm run lint                       # ESLint (TS/TSX/JS/JSX)
npm run lint:fix                   # ESLint auto-fix
npm run format-docs                # Prettier for JSON/Markdown
npm run test                       # Run all Jest tests
npm run test:watch                 # Jest watch mode
npm run test:coverage              # Jest with coverage
npx jest <file> --no-coverage      # Run a single test file (e.g., src/__tests__/xxx.test.ts)
```

## Architecture Overview

This is a React Native clipboard synchronization app built with **Expo SDK 55**, using React Native 0.83 and React 19. It syncs clipboard content (text, images, files) between devices via SyncClipboard Server, WebDAV, or S3 backends.

**Current status: Android-only. iOS support is planned for the future.**

All code should be written with cross-platform portability in mind (see [Cross-Platform Design Principles](#cross-platform-design-principles) below). When writing code, consider iOS compatibility but do **not** implement iOS-specific features or native modules right now — just avoid patterns that would make iOS support unnecessarily difficult later.

### Three Entry Points

`index.ts` registers three separate RN roots, each serving a distinct purpose:

| Registration     | Component                    | Purpose                                                                              |
| ---------------- | ---------------------------- | ------------------------------------------------------------------------------------ |
| `main`           | `App.tsx`                    | Full app UI with bottom tab navigation (Home / History / Settings)                   |
| `quickAction`    | `src/QuickActionApp.tsx`     | Transparent overlay for quick-settings tiles, share menu, and text selection actions |
| `serviceRestart` | `src/ServiceRestartApp.tsx`  | Brief "service restored" screen shown when Android restarts the process              |
| Headless         | `src/tasks/SmsUploadTask.ts` | Background SMS verification code upload (no UI)                                      |

`App.tsx` handles cold/hot start deep link routing: parses `syncclipboard://` URLs to determine whether to show the main UI or an overlay (share receive, quick upload/download).

### State Management

**Zustand** stores in `src/stores/`:

- `useSettingsStore` — central app config (servers, sync settings, clipboard access methods, UI preferences). Persisted via `ConfigService` → `AsyncStorage`.
- `useLocalClipboardStore` — current local clipboard content
- `useHistoryStore` — clipboard history items
- `useMessageStore` / `useErrorStore` — toast messages and errors (UI-only, not persisted)
- `useClipboardSyncServiceStore` (in `src/serviceState/`) — sync operation state (progress, remote content, upload/download flags)

### Service Layer

Key singletons, typically accessed via `getInstance()` or module-level exports:

- **`ConfigService`** (`src/services/ConfigService.ts`) — the single source of truth for persisted app configuration. Reads/writes `AsyncStorage`. Has a subscriber pattern for change notification.
- **`ClientFactory`** (`src/services/ClientFactory.ts`) — creates the appropriate API client from the active server config. Supports three backends:
  - `SyncClipboardClient` — custom server (HTTP + optional SignalR)
  - `WebDAVClient` — WebDAV storage
  - `S3Client` — S3-compatible object storage
    Clients implement `ISyncClipboardAPI` (in `src/api/clients/APIClient.ts`), which uses `axios` + `AuthService`.
- **`BackgroundRuntimeState`** (`src/services/BackgroundRuntimeState.ts`) — non-persisted boolean flag (`isTempDisabled`) for temporarily disabling background tasks. Decoupled from Zustand to avoid circular dependency between `LongRunningTaskManager` and `settingsStore`.
- **`ClipboardMonitor`** (`src/services/clipboard/ClipboardMonitor.ts`) — polls the system clipboard for changes using `native-timer` (not `setInterval`, to survive background). Detects changes via hash comparison. Accepts background-running checkers.
- **`RemoteClipboardMonitor`** (`src/services/sync/RemoteClipboardMonitor.ts`) — monitors the remote clipboard. Uses SignalR for SyncClipboard servers, polling for WebDAV/S3. Has deduplication via `DedupedOperation` and content hash tracking.
- **`ClipboardSyncService`** (`src/services/sync/ClipboardSyncService.ts`) — wires together local clipboard monitoring, remote monitoring, and history sync. Subscribes to clipboard changes, remote changes, history changes, and transfer queue events.

### LongRunningTask Framework


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jeric-X/syncclipboard-mobile](https://github.com/Jeric-X/syncclipboard-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
