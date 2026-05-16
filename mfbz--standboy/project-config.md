---
trigger: always_on
description: Standboy is a VSCode/Cursor extension that auto-expands a Game Boy emulator panel while an AI coding agent is generating, and auto-collapses when it stops. The product is the shell + behavior — agent-activity detection, the auto-show/hide UX, brand chrome — wrapped around a third-party emulator (EmulatorJS).
---

# CLAUDE.md

## Project

Standboy is a VSCode/Cursor extension that auto-expands a Game Boy emulator panel while an AI coding agent is generating, and auto-collapses when it stops. The product is the shell + behavior — agent-activity detection, the auto-show/hide UX, brand chrome — wrapped around a third-party emulator (EmulatorJS).

**Read first:** `SPECS.md` (product spec), `design.html` (visual identity), `media/icon.svg` (canonical icon; rendered to `media/icon-{128,512,1024}.png`).

## Tech stack

- TypeScript (strict), VSCode Extension API ≥1.95, esbuild (extension host CJS + webview IIFE).
- React 19 + Tailwind v4 in the webview. `lucide-react` for iconography.
- EmulatorJS as npm deps (`@emulatorjs/emulatorjs` + `core-gambatte` + `core-mgba`). `scripts/vendor-emulatorjs.mjs` copies from `node_modules/@emulatorjs/*` into `vendor/emulatorjs/` before every `compile` / `build` / `watch`. `vendor/` is gitignored but ships in the `.vsix`.
- **No-Intro ROM database** (`scripts/build-rom-db.mjs`) fetches DATs from a pinned commit of `libretro/libretro-database`, parses to compact `sha1 → canonical name` JSON. Output at `data/rom-db/{gb,gbc,gba}.json` is **committed** (not gitignored, unlike `vendor/`) so builds are deterministic and offline. Refresh by bumping the SHA in the script and running `npm run rom-db`; commit the resulting JSON diff alongside the SHA bump.
- Vitest, Prettier, ESLint, GitHub Actions (build / format / test / release).

## Architecture

- Single-package flat repo. Extension host code under `src/` (Node), webview code under `webview/` (browser). Both share `src/messages.ts` as the postMessage type contract.
- Palette via four CSS custom properties (`--sb-c0..c3`). Switching palette updates the four variables — no React re-render of chrome.
- **ROM library (`src/library.ts`).** All persistent user data lives under one root, pointable at iCloud/Dropbox/etc. via `standboy.libraryDirectory`:
  - `<libraryRoot>/library.json` — index. Each entry: `name` (original filename), `canonicalName?` (No-Intro from rom-db), `ext`, `size`, `addedAt`, `lastPlayedAt`. `lastPlayedHash` at top level for auto-resume.
  - `<libraryRoot>/config.json` — user settings (key bindings today; future global prefs). Versioned envelope.
  - `<libraryRoot>/roms/<hash>.<ext>` — ROM file, content-addressed by SHA-256 first 16 hex (rename-resilient).
  - `<libraryRoot>/saves/<hash>.sav` — battery save mirror.
  - `<libraryRoot>/covers/<hash>.png` (or `.miss` marker) — fetched libretro box art.
- **Hash-based ROM identification (`src/database.ts`).** On import, the full SHA-1 of ROM bytes is matched against the bundled No-Intro DBs. Hits give us the canonical filename, used for (a) friendly display title (`friendlyName()` strips region/revision tags) and (b) the cover lookup against libretro-thumbnails. ROMs not in the DB (homebrew, hacks) fall back to the user's filename. `extension.ts` runs a one-time `backfillCanonicalNames()` on activate to retroactively identify pre-database imports.
- **Save persistence.** EmulatorJS auto-persists SRAM to the browser's IndexedDB via IDBFS (`autoPersist: true` in `vendor/emulatorjs/data/src/GameManager.js`) — that's why "Continue" works even if our disk file is empty. Our `<hash>.sav` is a portable mirror, written **event-driven, no polling** — on `visibilitychange→hidden`, `pagehide`/`beforeunload`, EmulatorHost cleanup, and force-flushed before Export Save / Import Save (via `window.__standboyFlushSave`). The extension serializes all webview→host messages through a single promise chain so a save's `writeFile` always completes before the next dependent read (auto-resume metadata, save round-trips, etc).
- **ROM byte delivery.** ROM bytes do **not** travel through `postMessage`. A 32MB GBA ROM serialised via `Array.from(uint8)` produced a ~128MB smi array plus a JSON string of similar size and would OOM the extension host on import. Instead, `loadAndPostRom` (`src/extension.ts`) `access()`-checks the ROM file at `library.romFilePath(hash, ext)`, builds a webview-resource URI via `provider.asWebviewFileUri`, and posts only the URI — `<libraryRoot>` is already in `localResourceRoots` so the webview can `fetch()` the bytes directly into a `Blob` and hand EJS a `blob:` URL. Saves stay inline (≤128KB, harmless). The webview revokes the blob URL inside `EJS_onGameStart` so the ROM-sized Blob is released as soon as EJS has copied it into its Emscripten FS.
- **Cover fetcher (`src/covers.ts`).** Tries the canonical name first, then progressively-stripped variants of the user's filename. Network calls happen in the extension host; webview only loads cached files via `asWebviewUri`. CSP stays locked-down. Concurrency 4, `coverUpdate` messages stream back to the grid as art lands.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mfbz/standboy](https://github.com/mfbz/standboy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
