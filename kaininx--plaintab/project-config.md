---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

PlainTab is a Chrome/Edge extension (Manifest V3) and standalone web page that replaces the browser's new tab page with a minimal, zero-flash wallpaper experience. Zero external dependencies, no build step — pure vanilla JS + CSS.

## Architecture

### Dual-layer wallpaper system (zero-flash)

Two stacked `<div>` layers in `index.html`:

- **`#wallpaperBack`** (z-index: 0) — always holds a visible image. `preload.js` synchronously writes the pre-formatted thumbnail via `back.style.backgroundImage` before the browser's first paint, so the user never sees a blank/gray background.
- **`#wallpaperFront`** (z-index: 1, `opacity: 0`) — used for fade-in transitions. New full-res images are preloaded in memory (`img.decode()`), set as `background-image` on the front layer, then faded in via CSS `opacity` transition. After the transition completes (~550ms), the image is "stabilized" onto the back layer (direct `back.style.backgroundImage` assignment) and the front layer resets to transparent.

This ensures at least one layer always holds a rendered image — no frame is ever blank.

### JS file loading order (critical)

1. **`js/preload.js`** — synchronous IIFE, runs before anything else. Reads `localStorage.ptab_mode` then either `ptab_img_order[idx]` → `ptab_img_thumbs[id]` (local) or `ptab_bing_thumb` (Bing). Must be in `<head>` or immediately after `#wallpaperBack` in the DOM.
2. **`js/languages.js`** — defines `window.I18N` (16 locales) and `window.LanguageList`. Must load before `newtab.js`.
3. **`js/newtab.js`** — the main application (~945 lines). IIFE with clear sections: Constants, Environment detection, DOM refs, State, Utils, i18n, IndexedDB storage, Wallpaper (Bing fetch, local multi-image upload with rotation, dual-layer apply), UI (panels, corner buttons, search bar, local gallery), Extension mode override, Bootstrap.

### Two runtime modes

`newtab.js` detects its environment at init via `typeof chrome !== 'undefined' && chrome.runtime && !!chrome.runtime.id`:

- **Extension mode** (`chrome.runtime.id` exists): uses `chrome.search.query()` for CWS single-purpose compliance. The engine selector UI row is hidden and the engine icon becomes a static magnifying glass. Search uses the browser's built-in default search engine.
- **Web mode** (Cloudflare Workers / GitHub Pages): full engine selector (Google → Bing → Baidu → DuckDuckGo). Search opens `window.open(url, '_self')`.

### Storage

- **IndexedDB** (`PlainTab`, v1, store `wallpaper`): stores raw image blobs.
  - `ptab_bing_blob` — single Blob for the Bing daily wallpaper
  - `ptab_img_<id>` — per-image blob entries: `{blob, mime, name}` for each user-uploaded wallpaper (max 12). Each image is its own IDB key, allowing single-key read/write/delete without touching other images
- **localStorage**:
  - `ptab_version` — data schema version (`2`), for future migration detection
  - `ptab_bing_thumb` — single thumbnail data URL (CSS-ready `url(data:...)`), written by `applyWallpaper()` only in Bing mode, read by `preload.js`
  - `ptab_img_order` — JSON array of image IDs `["id1","id2",...]`, determines rotation order. Single source of truth for which images exist
  - `ptab_img_thumbs` — JSON object `{id1: "url(data:...)", id2: ...}`, id→thumb map. Lookup via `thumbs[order[idx]]`, never by array index
  - `ptab_local_index` — rotation index for local wallpapers, incremented each new tab, modulo order.length
  - `ptab_bing_meta` — `{src, date, provider}` for Bing image dedup and freshness checks. `src` doubles as the dedup key
  - `ptab_mode` — `'bing'` or `'local'`
  - `ptab_lang`, `ptab_search_mode`, `ptab_icon_opacity`, `ptab_search_engine` — UI preferences

**Crash consistency design (v2):**
- **Upload**: write blob to IDB first (`ptab_img_<id>`), then update order + thumbs in localStorage. If crash before order update, blob is orphaned and safely ignored
- **Delete**: remove id from order first, then delete thumb from map, finally delete IDB blob. If crash after order change, blob is unreachable but harmless
- **Gallery**: reads all `ptab_img_<id>` in parallel via `Promise.all`, fallback to blob URL only if thumb is missing

**Version bump rules:**
- **localStorage keys added, removed, or renamed** → increment `LS_VERSION` in `newtab.js` and write migration logic
- **IDB store schema changed** (new/removed store, index changes) → increment `DB_VERSION` in `newtab.js`
- **IDB key name changed** (same store, just the key string) → no version bump needed; IDB keys are opaque strings
- `ptab_version` itself must never be renamed or removed

### Multi-image local wallpaper

Users can upload up to 12 local wallpapers. Each new tab rotates to the next image via `ptab_local_index` modulo `ptab_img_order.length`.

**Upload flow**: `saveLocalImage(file, show)` generates thumbnail via canvas, writes blob to IDB as `ptab_img_<id>` (single key), then appends id to `ptab_img_order` and thumb to `ptab_img_thumbs[id]`. Blob is written first — if crash before order update, orphan blob is safely ignored.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaininx/PlainTab](https://github.com/kaininx/PlainTab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-11 -->
