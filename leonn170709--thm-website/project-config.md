---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
node server.js        # Start the server (port 3002)
npm install           # Install dependencies (express, basic-auth)
```

No build step, no bundler, no test suite.

## Architecture

Static site served by a minimal Express server (`server.js`). Everything under `public/` is served as-is.

```
public/
  index.html                 # The landing page
  css/thm.css                # The entire design system, shared by every page
  js/thm.js                  # All behaviour, shared by every page
  js/skin.js                 # 3D Minecraft skin viewer, shared by profile pages
  players/<Name>/index.html  # Per-member profile pages
  players/Kyr0zen/art/       # Kyr0zen's portfolio images
  images/                    # .webp assets (banner, logo, map, kitbot icon)
  stats.json                 # Served live via GET /api/stats
  map-meta.json              # Highway map's last-updated timestamp
```

`server.js` does four things: `express.static('public')`, a `GET /api/stats`
endpoint that reads `public/stats.json` with `Cache-Control: no-store` so
Cloudflare can't cache it, `GET /cape/index`, and PNG→WebP. It retries on
`EADDRINUSE` up to 5 times with exponential backoff.

`GET /cape/index` is the cape index for the THM-Addons mod (its `CapeManager`
fetches it as `api.capeIndex`). It answers `{"capes":[{"id","url"}]}`, one entry
per `.webp` in `public/cape/`, `id` being the filename without the extension —
the mod saves each download as `<id>.webp`. Adding a cape is dropping a `.webp`
in that directory; nothing else to edit. The `url` is absolute and built from
`x-forwarded-proto`, since the client is a Java `HttpURLConnection` that won't
follow Cloudflare's http→https redirect.

PNG→WebP: on boot, every `.png` under `public/` gets a lossless `.webp` twin via
system **ffmpeg** (`-c:v libwebp`), skipped when the twin is already newer than
the PNG. A middleware then answers `.png` requests with the twin when the client
sends `Accept: image/webp` and the twin is smaller, `Vary: Accept` set. No image
library in the dependencies; a new PNG converts on the next restart. If ffmpeg is
missing the conversion warns and the PNG is served as-is.

`stats.json` and `map-meta.json` are written by something outside this repo. If
either fetch fails the page degrades quietly (the plate keeps its `—`
placeholders).

## Frontend conventions

No framework, no build. Plain HTML, one shared stylesheet, one shared script.
CSS and JS are **not** inline. Each `<head>` carries exactly two inline
`<script>`s and nothing else: the pre-paint theme setter (which also sets
`style.colorScheme`, so a new document never flashes white), and a
`type="speculationrules"` block — JSON, not behaviour.

### Navigation is client-side

**A same-origin click never loads a document.** `thm.js` intercepts it, fetches
the page, and replaces `<body>` — so nothing blanks, in any browser. Chrome and
Safari wrap the swap in `document.startViewTransition` and crossfade it; Firefox
gets an instant swap. This is the only way to match a site that stays smooth in
Firefox, which has neither view transitions nor speculation rules.

The rule this imposes on all JS: **`thm.js` runs its page wiring more than
once.**

- `initPage()` — everything that touches page content. Runs on load and after
  every swap. It disconnects the previous page's `IntersectionObserver`s first
  (`track()` collects them); anything else stateful must be equally re-entrant.
  The nav is part of the swapped body, so the theme toggle and burger are wired
  here too.
- `initShell()` — window-level listeners only (scroll, the router). Runs once.
- Scripts inside swapped-in markup **never execute**. That is why pages carry no
  `<script>` for three.js or `skin.js`: `skinViewer()` loads them on demand, once
  per session, when a `#skin-canvas` is present. `skin.js` is therefore
  `window.thmSkin()`, not an IIFE, and its render loop stops itself when the
  canvas is detached — otherwise every visited profile leaks a WebGL context.
- Failure hands control back: a bad fetch or an empty document sets
  `location.href`, so a broken page is a normal page load, never a dead end.
- Without JS every link is an ordinary link, which is why the rest still matters.

**No page may show a white frame on a direct load either.** Three pieces, and a
page missing any one of them flashes:

1. **The pre-paint script sets the background**, not just the theme:
   `r.style.background = t === 'light' ? '#f3efe8' : '#0d0b0c'` (the two `--void`
   values). It stays inline — an external file would be a blocking request in
   front of first paint, which is the thing being fixed. `<meta
   name="color-scheme">` and `color-scheme` in `thm.css` back it up but cannot
   replace it: `"dark light"` resolves to **light** on a light desktop and paints
   white even when the visitor picked the dark theme. The theme toggle sets the
   same three properties, or the next swap paints the old theme for a frame.
2. `@view-transition { navigation: auto; }` in `thm.css`, **at top level**, once.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Leonn170709/THM-Website](https://github.com/Leonn170709/THM-Website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
