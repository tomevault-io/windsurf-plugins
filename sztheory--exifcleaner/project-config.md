---
trigger: always_on
description: Cross-platform Electron desktop app to strip EXIF/metadata from images, videos, and PDFs. Wraps bundled `exiftool` Perl binaries. MIT license.
---

# ExifCleaner

Cross-platform Electron desktop app to strip EXIF/metadata from images, videos, and PDFs. Wraps bundled `exiftool` Perl binaries. MIT license.

## Tech Stack

- **Runtime**: Electron 35 (Chromium + Node 22) with contextIsolation + sandbox
- **Language**: TypeScript 5.7 with `strict: true` + `verbatimModuleSyntax: true` (type-check only, electron-vite/esbuild compiles)
- **Build**: electron-vite 5.x + Vite 7.x + esbuild (3 targets: main, preload, renderer)
- **Packaging**: electron-builder 22.8 (produces .dmg, .AppImage, .deb, .rpm, .exe, portable)
- **UI**: React 19 SPA with BEM CSS design system
- **ExifTool**: Hand-rolled wrapper in `src/infrastructure/exiftool/` wrapping bundled exiftool Perl binaries
- **Formatting**: Prettier 3.x with tabs
- **Dependencies**: Three production dependencies (react, react-dom, zod) — ExifTool wrapper is hand-rolled
- **Performance**: Processing speed is a core product value — the app must handle hundreds of files in seconds. Never add latency to the file processing pipeline.

## Commands

```bash
yarn dev              # Dev mode with HMR (electron-vite dev server)
yarn dev:debug        # Dev mode + Chrome DevTools Protocol on port 9222 (for MCP)
yarn compile          # Build with electron-vite (esbuild)
yarn format           # Format code with Prettier
yarn lint             # Check formatting with Prettier

yarn packmac          # Build macOS .dmg (unsigned)
yarn packwin          # Build Windows .exe (NSIS + portable)
yarn packlinux        # Build Linux (.AppImage, .deb, .rpm)
yarn build            # Build all platforms
yarn publish          # Build all + publish to GitHub releases

yarn run update-exiftool  # Update exiftool binaries (requires Perl, Linux/Mac only)
```

## Debugging

Chrome DevTools MCP is configured for AI-assisted debugging of the Electron app:

1. Run `yarn dev:debug` (launches app with `--remote-debugging-port=9222`)
2. Claude Code connects via the `chrome-devtools` MCP server to read console logs, take screenshots, evaluate JS

MCP config is in `.claude.json` (project-scoped). The server uses `chrome-devtools-mcp` npm package with `--browser-url=http://127.0.0.1:9222`.

## Architecture

Three-process Electron model: main, preload (contextBridge), renderer (sandboxed browser). Shared `common/` layer.

### Main Process (`src/main/`)

Entry: `index.ts` → `init.ts` (i18n, exif handlers, context menu, dock, app handlers) → `window_setup.ts` (BrowserWindow creation)

- `app_setup.ts` — single instance lock, lifecycle events, exiftool cleanup on quit
- `dock.ts` — IPC handlers for progress tracking, Mac dock badge, Windows taskbar flash
- `exif_handlers.ts` — exiftool IPC handlers (`exif:read`, `exif:remove`), uses hand-rolled `ExiftoolProcess` from infrastructure layer
- `file_open.ts` — native file dialog
- `menu*.ts` — menu bar templates (app, file, edit, view, window, help, dock)
- `i18n.ts` — main process i18n, exposes locale and strings via IPC

### Preload Script (`src/preload/`)

Bridge between isolated renderer and Node.js. Only file that uses `contextBridge`.

- `index.ts` — exposes `window.api` with exif, i18n, and files namespaces
- `api_types.ts` — TypeScript interfaces for the contextBridge API

### Renderer Process (`src/renderer/`)

Entry: `index.ts` → async setup (i18n, drag-drop, file selection menu). **Fully sandboxed** — no Node.js, no Electron imports. All external access through `window.api.*`.

- `drag.ts` — drag-and-drop event listeners on `document` (file.path is an Electron extension that persists with contextIsolation)
- `select_files.ts` → `add_files.ts` — core processing pipeline:
  1. For each file: add row → `window.api.exif.readMetadata()` → `window.api.exif.removeMetadata()` → read after → update row
  2. Notifies main process via `window.api.files.notify*()` for dock badge / progress bar
- `display_exif.ts` — coordinates before/after EXIF display via `window.api.exif`
- `table_add_row.ts` / `table_update_row.ts` — DOM table manipulation
- `sanitize.ts` — XSS prevention: uses `innerText` not `innerHTML` for exiftool output
- `selected_files.ts` / `empty_pane.ts` — UI state management
- `i18n.ts` — fetches strings via `window.api.i18n`, caches locally, uses pure `i18nLookup()` from common
- `env.d.ts` — ambient `window.api` type declaration

### Common (`src/common/`)

Shared between processes (main imports Node-dependent files, renderer imports pure files only):

- `binaries.ts` — resolves platform-specific exiftool binary path (main only)
- `i18n.ts` — loads `.resources/strings.json`, delegates to `i18n_lookup.ts` (main only)
- `i18n_lookup.ts` — pure i18n lookup: Locale enum, fallback logic, no Node.js deps (safe everywhere)
- `platform.ts` — `isMac()`, `isWindows()`, `isLinux()` helpers
- `browser_window.ts` — safe BrowserWindow reference helpers
- `resources.ts` — resource path resolution (dev vs production)
- `env.ts` — `isDev()` detection
- `ipc_events.ts` — IPC channel name constants

### Infrastructure (`src/infrastructure/`)

Hand-rolled wrappers for external processes (main process only):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [szTheory/exifcleaner](https://github.com/szTheory/exifcleaner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
