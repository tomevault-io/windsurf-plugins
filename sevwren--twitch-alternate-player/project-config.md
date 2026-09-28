---
trigger: always_on
description: This file provides persistent instructions for Claude Code agents working in this repository.
---

# CLAUDE.md — Twitch Alternate Player

This file provides persistent instructions for Claude Code agents working in this repository.
Read it fully before making any changes.

---

## Agent Start Here

**Three facts every agent must hold before touching any file:**

1. All identifiers in this codebase are in **Russian (Cyrillic)**. New code must use **English only**.
2. `Documentation/legacy_code_translation_reference.md` is the authoritative Russian→English glossary.
3. `.agent/rules/js-doc-master-template.md` defines mandatory JSDoc format for translated functions.

---

## Project Ownership & Context

**The current maintainer does not speak Russian.**

This project was originally developed by a Russian-speaking developer (Alexander Choporov / CoolCmd).
Variable names, function names, constants, CSS classes, HTML IDs, comments, and some UI strings are all Cyrillic.
The current maintainer is actively translating **everything** to English.

- Introduce no new Cyrillic identifiers or strings under any circumstances.
- When modifying existing code, translate Cyrillic identifiers in the surrounding scope as part of the change (opportunistic translation).
- Translation commits must be **atomic per file** — no mixing translation with logic changes.
- Cyrillic `chrome.runtime.sendMessage` event names must be renamed in **sender + listener pairs simultaneously**.

---

## Project Overview

**Alternate Player for Twitch.tv** — Chrome extension (Manifest V3) that replaces Twitch's native player:

- Server-side ad bypass (omits `play_session_id` to skip SSAI ad injection)
- Real-time stream diagnostics overlay (the **S** key statistics panel)
- WebAssembly-accelerated TS→MP4 transcoding via a Web Worker
- BetterTTV / FrankerFaceZ emote integration (generic CDN fallback)
- Auto-claim channel points, GraphQL token interception, duplicate-tab prevention

**GitHub:** https://github.com/SevWren/twitch_alternate_player
**Manifest version:** 3 | **Min Chrome:** 92 | **Extension version:** 2025.5.28

---

## Architecture

### File Roles

| File | Layer | Role |
|---|---|---|
| `manifest.json` | Config | Extension entry point — permissions, content scripts, web-accessible resources |
| `background.js` | Service Worker | Inter-tab messaging: detects third-party extensions, prevents duplicate player tabs |
| `content.js` | Content Script | Detects Twitch channel URLs, redirects to `player.html` |
| `gqltoken.js` | Content Script | Intercepts Twitch GraphQL integrity token at `document_start` |
| `autoclaim.js` | Content Script | Auto-claims channel bonus points |
| `content_injection.js` | Page Context | Injected into page context for GQL token capture (MV3 workaround) |
| `gql_injection.js` | Page Context | Additional GQL interception (web-accessible resource) |
| `player.html` | Player UI | Full alternate player — video element, controls, settings tabs, stats overlay |
| `player.js` | Player Logic | ~8,000-line main module: HLS fetching, MSE pipeline, stats, settings, chat |
| `player.css` | Styles | Player UI styling |
| `common.js` | Shared Util | Shared helpers: i18n wrapper, storage, DOM utilities |
| `worker.js` | Web Worker | MPEG-TS demuxer → MP4 muxer, runs off main thread |
| `wasm.wasm` | Binary | Compiled WebAssembly runtime for segment transcoding — **never open as text** |
| `asmjs.js` | Fallback | asm.js fallback when WASM unavailable |
| `rules.json` | Net Rules | `declarativeNetRequest` rules — strips ad-related request/response headers |
| `_locales/en/messages.json` | i18n | English UI strings |
| `_locales/ru/messages.json` | i18n | Russian UI strings |
| `recycler.js` | Util | Memory recycler — reuses typed-array buffers to reduce GC pressure |
| `pointerevent.js` | Util | Pointer event / drag utilities |
| `report.html` / `report.css` | UI | Extension report/feedback page |

### Message-Passing Flow

```
Twitch page
  └── content.js          (detects channel, redirects)
  └── gqltoken.js         (captures GQL integrity token)
  └── content_injection.js (page-context token relay)
        |
        v
  background.js           (service worker: tab dedup, ext detection)
        |
        v
  player.html + player.js (HLS fetch → MediaSource)
        |
        v
  worker.js               (TS demux → MP4 mux via WASM)
```

---

## Development Commands

There is **no build system**. This is a raw browser extension — edit files directly.

| Task | How |
|---|---|
| Load extension | Chrome → `chrome://extensions` → "Load unpacked" → select repo root |
| Reload after JS/HTML/CSS change | Click the reload icon on `chrome://extensions` |
| Reload after `manifest.json` change | Remove and re-add the extension |
| Reload background worker | Click "Service worker" link on the extension card |
| Debug player | Open `player.html` tab → DevTools → Console / Sources |
| Debug background | Click "Service worker" → DevTools |
| Capture logs | Run `node tools/chrome-tools/fetch_logs.js` (requires Chrome remote debugging on port 9222) |
| Git diff bundle (Windows) | Run `tools/git_changes_tool/gather_comparison_info.ps1` |

**Run the extension after every non-trivial change** — there are no automated tests.

---

## Code Style & Conventions

### Language & Identifiers


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SevWren/twitch_alternate_player](https://github.com/SevWren/twitch_alternate_player) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
