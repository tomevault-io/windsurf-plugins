---
trigger: always_on
description: Electron-based markdown editor built around Monaco. The same TypeScript bundle ships as a desktop app (Electron wraps it) and as a pure web app (deployed to GitHub Pages). Current version: 4.0.0.
---

# MKEditor — Project Context

Electron-based markdown editor built around Monaco. The same TypeScript bundle ships as a desktop app (Electron wraps it) and as a pure web app (deployed to GitHub Pages). Current version: 4.0.0.

The renderer is React 19 + shadcn/ui + Tailwind v4 on top of a set of plain-TS managers (Monaco, files, settings, markdown, IPC bridge). Migration history: [docs/REACT_MIGRATION.md](docs/REACT_MIGRATION.md). Subsystem READMEs: [src/app/README.md](src/app/README.md), [src/browser/README.md](src/browser/README.md), [src/browser/extensions/README.md](src/browser/extensions/README.md). For deeper detail see [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md). For planned work see [docs/ROADMAP.md](docs/ROADMAP.md).

## Repo Layout

- [src/app/](src/app/) — Electron main process (Node). Compiled by `tsc` via [scripts/compile-app.mjs](scripts/compile-app.mjs) to `dist/app/`.
- [src/browser/](src/browser/) — Renderer (Monaco + React). Bundled by webpack to `dist/mkeditor.bundle.js` + `dist/mkeditor.bundle.css`.
- [locale/](locale/) — i18next JSON resources, one folder per language. Build step combines per-namespace JSON into `all.json` via [scripts/combine-locales.mjs](scripts/combine-locales.mjs).
- [tests/](tests/) — Jest + jsdom. Manager-level suites at the root; React component tests (RTL) under `tests/react/`. Mocks for `electron`, `monaco-editor`, CSS imports under [tests/**mocks**/](tests/__mocks__/).
- [@types/index.d.ts](@types/index.d.ts) — Global window augmentations (`window.executionBridge`, `window.mked`, `window.logger`, `window.setLanguage`).
- [build/](build/) — Installer resources (icons, license).

## Two Runtime Modes

The same renderer bundle detects which mode it's running in via [getExecutionBridge()](src/browser/util.ts):

- **Desktop**: `window.executionBridge` is injected by the Electron preload ([src/app/preload.ts](src/app/preload.ts)). Three sibling JSON files persist under `~/.mkeditor/`: `settings.json` (editor + export preferences), `session.json` (tabs, active tab, workspace folder, per-tab cursor/scroll/folding — restored on relaunch), and `assistant.json` (AI Assistant config + conversation history; API keys live in its `keys` section, encrypted with Electron `safeStorage` and never crossing IPC in plaintext). File tree sidebar visible. Files open through IPC.
- **Web**: no bridge. Settings + last-edited content go to `localStorage` (`mkeditor-settings`, `mkeditor-export-settings`, `mkeditor-content`). Sidebar collapsed by default, "delete content" button shown in the toolbar. Exports use the File System Access API or `window.open` + `print()` for PDF.

Mode branching lives in [index.ts](src/browser/index.ts), [EditorManager](src/browser/core/EditorManager.ts), [SettingsProvider](src/browser/core/providers/SettingsProvider.ts), [ExportSettingsProvider](src/browser/core/providers/ExportSettingsProvider.ts), and a few React components that conditionally render desktop-only chrome.

## IPC Bridge Model

Channels are whitelisted in [preload.ts](src/app/preload.ts:15-53):

- **Renderer → Main** (`to:*`): `to:title:set`, `to:editor:state`, `to:settings:save`, `to:html:export`, `to:pdf:export`, `to:file:*`, `to:folder:*`, `to:i18n:set`, `to:ai:*` (config get/set, key set/clear, chat, cancel, tool-result, conversations save/flush, ollama list).
- **Main → Renderer** (`from:*`): `from:theme:set`, `from:settings:set`, `from:file:*`, `from:folder:*`, `from:modal:open`, `from:command:palette`, `from:notification:display`, `from:path:*`, `from:i18n:set`, `from:ai:*` (config, chunk, tool-call, done, error, ollama:models, conversations, conversations:flush-request), `from:assistant:toggle`.

Additionally `window.mked` exposes synchronous/invoke helpers for the `mked://` link provider, app locale, and path resolution. `window.logger` forwards renderer logs to `electron-log` via an `ipcMain.on('log', …)` handler in [AppBridge.ts](src/app/lib/AppBridge.ts).

Notifications cross the bridge as `{ status, key, values? }` (i18n) or `{ status, message }` (plain). [BridgeListeners.ts](src/browser/core/BridgeListeners.ts) translates and surfaces them via `sonnerToast` (the sonner `<Toaster />` in `<App>`).

## Renderer Composition Root

[src/browser/index.ts](src/browser/index.ts) wires the system at boot:

1. Register FontAwesome icons ([icons.ts](src/browser/icons.ts)) and pre-build the (now tiny) i18n binding for the splash `<h1>` ([i18n.ts](src/browser/i18n.ts)).
2. Initialise i18n with the user's locale (prefetch combined bundle, set `<html lang>` early, apply translations on load).
3. Construct an [EditorDispatcher](src/browser/events/EditorDispatcher.ts) and an [EditorManager](src/browser/core/EditorManager.ts) — Monaco is **not** created yet; that's `<EditorHost>`'s job inside the React tree.
4. `createRoot(#react-root).render(<App initialManagers onEditorReady />)` mounts the full provider tree synchronously, including the `<ModalsBridge>` / `<PromptsBridge>` / `<PropertiesBridge>` sentinels that install module-level seams.
5. `<EditorHost>`'s `useEffect` calls `editorManager.create({mount, watch:true})` once, then fires `onReady()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [versyxdigital/mkeditor](https://github.com/versyxdigital/mkeditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
