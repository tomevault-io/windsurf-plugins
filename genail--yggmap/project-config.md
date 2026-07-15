---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**yggmap** (live at https://yggmap.com; storage keys deliberately keep the older `mindmaps.` prefix — renaming them would orphan users' data): a free, fully private, one-page mind-mapping web app in the Excalidraw spirit — canvas-is-the-app, local-first, keyboard-first, zero onboarding.

## Commands

There is **no build system or linter — on purpose**. Plain HTML/CSS/JS; the
`package.json` npm tooling exists only for the test suite and never ships.

```sh
python3 -m http.server 8471     # serve for development; file:// won't work (localStorage)
# → http://localhost:8471/index.html

npm test                        # headless Playwright suite in tests/ (one-time
                                # setup: npm install && npx playwright install chromium)

npx wrangler deploy             # deploy to production (yggmap.com, Cloudflare Workers
                                # static assets; one-time auth: npx wrangler login)
```

Deployment serves the repo root as-is; `.assetsignore` lists what never ships
(mockups, `assets/src` logo sources, docs, test tooling) — extend it when
adding dev-only files. Deploying is a production action: **only deploy when
the user asks.**

### Tests (`tests/`)

`npm test` drives the real app headless via `@playwright/test` (config in
`playwright.config.js`; it starts the http server itself, or reuses one already
running on 8471). Fixture maps are planted straight into `localStorage` by
`tests/helpers.js#seedMap` before page scripts run — the seed is idempotent
across reloads so persistence tests can assert the app's own saves. Each test
gets a fresh browser context, so storage starts clean for free. **A bug fix or
behavior change should come with a test here** (red/green when practical).
CI runs the suite on every push and PR (`.github/workflows/tests.yml`).

Exploratory verification still happens by hand in a browser (Playwright MCP). Practical notes for both:
- Bust caching by appending `?v=<anything>` to the URL after editing files.
- `window.__mm()` returns `{focusId, editingId, rootId, currentMapId, undoDepth}` for assertions.
- Glide/pan animations use `requestAnimationFrame`; in a **background** tab they crawl due to rAF throttling — a test artifact, not a bug. Time-based easing still converges.
- A live page re-saves state on unload (blur → commit → save), so wiping `localStorage` must happen before navigation, not on the page under test.
- In-app **debug mode**: `⌘⇧D`/`Ctrl+Shift+D`, persisted in the UI storage key. Currently reveals the 📱 mobile-preview button that forces the touch UI on desktop.

## Architecture

Four files, no modules, load order matters (`sketch.js` then `app.js`, both global-scope):

- `index.html` — app shell + shell-only CSS (positioning of fixed chrome, drawer/panel visibility). Component styling does NOT belong here.
- `styles.css` — the living design system: all `.mm-*` components and tokens. Shared by the app, `book.html`, and some mockups. **Any new/changed component must also be represented in `book.html`** (the component book is documentation-by-rendering).
- `sketch.js` — `MMSketch`: seeded-PRNG hand-drawn SVG connectors (jittered bezier + faint offset "pencil echo"). Seed comes from node-id hashes so lines don't rewiggle every render.
- `app.js` — everything else, organized in commented sections.

### Render pipeline (app.js)

`refresh()` runs the whole pipeline on every mutation:

```
doc (JSON tree) → buildShadow()   clone of visible tree with ghost entries injected
               → layoutShadow()   DOM-measure sizes → subtree-height stacking (H_GAP/V_GAP)
               → renderShadow()   reconcile DOM nodes in #nodes (className composition)
               → applyPositions() absolute left/top; lastPos = world-coord centers
               → drawLinks()      SVG connectors in #links
               → renderChrome()   body classes, hint strip, mobile action bar
```

Key ideas that span multiple functions:

- **Ghosts are real layout citizens.** `ghost:child` / `ghost:sibling` placeholders show what Tab/Enter would create and the map genuinely makes room for them. On desktop they're educational hints that fade with use (`learnLevel` capped at 2 — faint but **never fully hidden**; silent disappearance reads as a bug). On mobile they ARE the create buttons. Only real Tab/Enter keypresses increment learn counters — never taps.
- **Sticky sides.** Depth-1 branches persist `node.side` ('R'/'L'); new branches go to `lighterSide()`, siblings inherit. Prevents whole-branch reshuffling when the layout changes.
- **Viewport is a transform** on `#world` (`view = {x, y, s}`). `ensureVisible(id)` pans minimally (smooth `glideTo`) so the focused node *plus its ghosts* fit inside margins that account for chrome and the mobile keyboard (`visualViewport`); it runs on focus change, edit start, **and commit** (ghosts only reappear after editing ends). User gestures cancel glides.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [genail/yggmap](https://github.com/genail/yggmap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
