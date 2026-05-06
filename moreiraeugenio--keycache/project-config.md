---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

Node.js **24.15.0** is pinned via `.nvmrc` (and softly enforced by `engines.node` in `package.json`). Run `nvm use` (or `fnm use`) before the commands below if your shell isn't already on the right version.

```bash
npm install                # install dependencies
npm run dev                # launch dev server with HMR (electron-vite dev)
npm run build              # production build to out/ (electron-vite build)
npm run lint               # ESLint on src/
npm run format             # Prettier on src/
npm run test               # unit tests (Vitest)
npm run test:e2e           # E2E tests (Playwright + Electron) — builds app, then runs
npm run package            # build + package unpacked app to dist/
npm run dist               # build + create distributable installer to dist/
npm run version:preview    # show bump level picked from commits since last tag (dry run)
npm run version:auto       # bump package.json + tag based on commits since last tag
npm run release            # full release flow: sync, checks, bump, push (one command)
npm run release:dry-run    # run all checks and preview, stop before bump + push
```

## Architecture

System tray desktop app for storing key-value notes in a local JSON file. TypeScript throughout, built with electron-vite. Lives in the menu bar (macOS) / notification area (Windows) / system tray (Linux). No native dependencies.

Follows Electron security best practices: `contextIsolation: true`, `nodeIntegration: false`, all data access via IPC in the main process.

```
src/
  main/
    index.ts          # App lifecycle, storage + settings init, tray/window/shortcut orchestration, settings IPC
    window.ts         # Frameless popup window: creation, positioning near tray, show/hide/toggle, blur + Escape auto-hide
    tray.ts           # System tray icon + context menu (Settings / About / Quit)
    shortcuts.ts      # Global shortcut registration (accelerator from settings)
    store.ts          # JSON file storage: in-memory notes with file persistence, NotesStore interface
    ipc.ts            # Notes IPC handlers (delegates to NotesStoreHolder.current — swappable)
    settings.ts       # AppSettings type, load/save (atomic .tmp+rename), moveDataFile (EXDEV-safe), defaults
  preload/
    index.ts          # contextBridge → window.api (notes, settings, dialog-open, hide, change subscriptions)
  renderer/
    index.html        # Single page with form / confirm / settings dialogs
    renderer.ts       # Orchestrator: list render, add/edit form, delete confirm, toast, search (debounced), value masking
    settings-dialog.ts  # Settings modal: theme, data file path browse, shortcut recorder, accelerator formatting
    shortcuts.ts      # In-window keyboard layer: ↑/↓ nav, Enter copy+hide, Escape, rebindable newNote / focusSearch
    styles.css        # Glassmorphism themes (dark + light via data-theme)
    env.d.ts          # Note / AppSettings / KeycacheApi type declarations
resources/
  trayIconTemplate.png      # macOS tray icon (22x22, template)
  trayIconTemplate@2x.png   # macOS retina (44x44)
  tray-icon.png             # Linux tray icon (256x256)
  tray-icon.ico             # Windows tray icon
tests/
  unit/                     # Vitest — 100% coverage enforced on src/main
    store.test.ts           # JSON storage CRUD + persistence tests
    ipc.test.ts             # IPC handler delegation tests (including NotesStoreHolder swap)
    index.test.ts           # App lifecycle + path resolution + settings IPC tests
    window.test.ts          # Window positioning + show/hide + blur + Escape tests
    tray.test.ts            # Tray icon path + context menu tests
    shortcuts.test.ts       # Global shortcut registration tests
    settings.test.ts        # loadSettings / saveSettings / moveDataFile tests
  e2e/
    app.test.ts             # Playwright — critical user flows
```

**Build output:** `out/` (electron-vite build) → `dist/` (electron-builder packaged app)

## Key Details

- **Storage:** Notes stored as JSON file (default `data.json`). All CRUD operates in-memory; file is written on every mutation. `createNotesStore(filePath)` returns a `NotesStore` interface. Handles missing/corrupted files gracefully (starts empty). The `NotesStoreHolder.current` reference in `ipc.ts` is swapped — not re-registered — when the user changes the data file path at runtime.
- **Settings:** Persisted next to the data file: `app.getAppPath()/settings.json` in dev, `app.getPath('userData')/settings.json` when packaged. `AppSettings` = `{ theme, dataFilePath, valuesHidden, shortcuts: { globalToggle, newNote, focusSearch } }`. Atomic write (`.tmp` + `rename`). `loadSettings` merges partial/corrupted files with defaults. `moveDataFile` prefers `rename`, falls back to `copyFileSync` + `unlinkSync` on cross-device errors (`EXDEV`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moreiraeugenio/keycache](https://github.com/moreiraeugenio/keycache) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
