---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Lens is an Electron desktop app that browses, searches, and resumes local Claude Code (`~/.claude/projects/*.jsonl`) and OpenAI Codex (`~/.codex/sessions/**/*.jsonl`) session history. The package name in `package.json` is `lens`.

## Common commands

```bash
npm install
npm run dev          # Vite (http://localhost:5173) + Electron in parallel
npm run typecheck    # tsc --noEmit (run before any commit)
npm run check:i18n   # scripts/check-i18n.mjs — diffs each locale vs en
npm run build        # typecheck + Vite build → dist/  (does NOT package)
npm run preview      # run Electron against built dist/

npm run dist:mac     # release/Lens-<ver>-(arm64|x64).zip
npm run dist:win     # release/Lens-<ver>-win.zip
npm run dist:linux   # release/Lens-<ver>-linux-x86_64.AppImage

npm run build:icon   # regenerate build/icon.{icns,ico,png} from icon.svg
DEMO_BUILD=1 npm run dist:mac   # screenshot-ready build with fake data forced on
```

There are no tests and no linter configured. `npm run build` runs `tsc --noEmit && vite build`. **`npm run check:i18n` is NOT in build** — running it reveals real gaps (non-English locales miss ~200 keys each); wire to CI only after you've committed to translating, otherwise it'll fail every build.

Renderer hot-reloads under `npm run dev`. **Main-process edits (`electron/main.cjs`) require a full restart** — there is no main-process reload. Session cache manifests (`sessions-cache-claude.json` / `sessions-cache-codex.json`, plus bounded shards when a source exceeds the single-file limit) under the platform userData directory survive across restarts. Parser/cache schema changes require a versioned migration in `electron/lib/sessions-cache.cjs`; do not rely on manually deleting caches.

## High-level architecture

### Two-process boundary

- `electron/main.cjs` (Node) — all filesystem IO, JSONL parsing, sub-process spawning (terminal launchers, `claude`/`codex` CLI probes), persistence (`favorites.json` / `excludes.json` / `aliases.json` / per-source session caches / `app-prefs.json` under `app.getPath('userData')`).
- `src/` (React 18 + TypeScript + Vite + Tailwind) — pure renderer. **No Node access.** Talks to main only via `window.api.*`.
- `electron/preload.cjs` — `contextBridge.exposeInMainWorld('api', ...)`. This is the IPC contract; any new feature that crosses the boundary needs an entry here + a matching `ipcMain.handle` in `main.cjs` + a typed signature in `src/types.ts` under `declare global { interface Window { api: { ... } } }`.

Electron window config in `createWindow()`: `contextIsolation: true`, `nodeIntegration: false`, `sandbox: false`. **Don't relax `contextIsolation` / `nodeIntegration`.** `sandbox: false` is intentional (preload uses `contextBridge` + `ipcRenderer`); flipping to `true` requires preload audit. Any new IPC handler that takes a path from the renderer must funnel through `ensureInside()` / `ensureInsideAny()` to prevent the renderer from reading files outside `~/.claude` / `~/.codex`. **Containment uses `isInsideBase(real, realBase)`** via `path.relative` — not lowercase prefix compare — so case-sensitive APFS volumes route correctly.

Renderer + main also enforce CSP (response-header injected, dev vs prod policies, dev origin derived from `VITE_DEV_SERVER_URL`), `will-navigate` refusing cross-origin, `setWindowOpenHandler` denying all and forwarding http(s)/mailto through `shell.openExternal`. Don't bypass these — add to the allowlist if a legitimate new origin appears.

### Single-instance lock

`app.requestSingleInstanceLock()` runs at startup; if a second Lens process launches, it focuses the existing window and quits. **All startup wiring (`whenReady`, `window-all-closed`, `activate`) lives inside the lock's `else` block** — a second instance must NOT register them, otherwise it briefly runs the full lifecycle before quitting.

### Provider registry — the source-aware pattern

`src/lib/sources.tsx` is **the** seam between "this app supports Claude Code" and "this app supports two AI tools". Every per-source thing — accent color, glyph, workspace blurb, kind labels ("Skills" vs "Rules"), path hints — lives in `SOURCES[id]`. Adding a future provider (Cursor, Gemini CLI, etc.) means one new entry in `SOURCES`, one in `SOURCE_ORDER`, a glyph component, and the main-process readers for that tool's on-disk shape. **Do not** add `source === 'codex' ? X : Y` ternaries inside components — read it off `getSource(currentSource)`.

`useCurrentSource()` is a singleton-backed hook (module-level `_current` + `_subs` set, mirrored to `localStorage['ai-source-v1']`). All views re-scope automatically when the user flips the sidebar provider switcher.

### Composite-key invariant (`source:id`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maliming/Lens](https://github.com/maliming/Lens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
