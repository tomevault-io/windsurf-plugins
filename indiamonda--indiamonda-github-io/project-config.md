---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A static **GitHub Pages** site (`indiamonda.github.io`, also reachable via `jimmyq-r-g.github.io`). One giant `index.html` hub links out to games, tools, unblockers, an in-page AI chat, and a GoGuardian-detection cloak. There is **no build step** — anything that runs in the browser ships as-is.

- **Owner:** `@jimmyqrg` (site owner — bypasses premium / unlimited features across all linked repos).
- **Production deploy:** branch `gh-pages`. **Commit only to `main`**; the owner pulls to `gh-pages` after verifying.
- **Backend for auth + cloud saves:** the separate `chat/` repo, deployed at `https://chat.jimmyqrg.com`.
- **AI chat proxy:** `cloudflare-worker/` (DeepSeek).
- **Sister repos mentioned by DEVELOPERS.md:** `chat/`, `../u/` (absolute unlinewize), `../q/`-style games live in a sibling `jg/g/` repo.

## Persistent agent memory

**`agent.md` at the repo root is the long-lived memory file.** It records task histories, gotchas, and per-game fixes. Update it before any chat-context compaction and re-read it when resuming. Do not delete or restructure it.

## High-level architecture

```
index.html            ← the hub. ~300KB / 5k+ lines. Game/app registry lives here.
                      All entries are base64-encoded via `_()` (decode with atob()).
                      Includes _D1.._D5 arrays for games, apps, unblocks, contacts, etc.

q/                    ← games hosted on this repo (each game is a folder with index.html)
q/e/, q/u/            ← other features routed off the hub

js/                   ← runtime browser scripts AND one-shot build/migration .mjs scripts
  jqrg-cloud.js       ← localStorage hijacker; syncs every write to chat.jimmyqrg.com
  jqrg-auth-ui.js     ← top-bar account button + sign-in/sign-up modal
  jqrg-gate.js        ← proxy detection; blocks the site if loaded through Ultraviolet/Scramjet/etc.
  jqrg-content-gate.js← reads auth state from localStorage; sets data-authed + __jqrgIsAuthed
  jqrg-particles.js   ← homepage background particles (5 styles × 4 quality tiers)
  jqrg-loader-lines.js← loading-screen tip pool
  openGame.js         ← window.openGame(url, sourcePage?) → loadGameInPage()
  panicKey.js         ← AltRight hotkey → panic URL (default: pausd.schoology.com)
  cursor.js           ← custom cursor renderer
  mainPageCloak.js    ← disguises non-game tab title/favicon (Gmail by default)
  gg-detect.js        ← GoGuardian extension detection → shows schoology-overlay.html
  ban-enforce.js      ← blocks specific emails/usernames, redirects to /tools/lagger/
  jqrg-aichat.js      ← floating AI chat widget (streams via cloudflare-worker/)
  educational-context.js ← JSON-LD decoy for AI scrapers (does not affect runtime)
  *.mjs               ← Node-only build/migration scripts (inject-cloud, migrate-loader-*,
                        audit-loader, strip-ads, fix-loader-newline, update-inject)

cloudflare-worker/    ← DeepSeek proxy. Allowed origins: indiamonda.github.io, jchat.fly.dev,
                        unlinewize.jimmyqrg.com, etc. See worker.js → ALLOWED_ORIGINS.

schoology-overlay.html ← 5.3MB fullscreen cloak shown when GoGuardian is detected
sw.js                 ← service worker (cache versioned `app-v{N}`)
css/                  ← main.css, info.css, tools.css, aichat.css
tools/                ← in-repo utilities (IndexedDB-reader, html-tester, lagger, …)
game-images/          ← game thumbnails (games/ and collections/)
cloak-images/         ← tab-cloak icons
loader/               ← loader assets
admin/                ← admin pages
api/                  ← static API stubs
about/, info/, learn/, join/, lx/, o/, strategies/, suggest-games/, html/, py/, IndexedDB-reader/
                      ← various feature sub-sites
67.html, nostalgia.html, schoology-overlay.html
                      ← standalone single-page apps
```

## Cloud saves / auth (most common integration)

Every same-origin HTML page is auth-gated. Both scripts are injected via a marker that build scripts can rewrite:

```html
<!-- JQRG_CLOUD_INJECT_BEGIN -->
<script src="/js/jqrg-cloud.js" defer></script>
<script src="/js/jqrg-auth-ui.js" defer></script>
<!-- JQRG_CLOUD_INJECT_END -->
```

After load, the global `JqrgCloud` exposes: `isLoggedIn / getUser / login / register / logout / forceSync / exportAll / importAll / deleteAll / snapshotIdb / restoreIdb / skipKey / skipKeys`.

- **New HTML page?** Run `node js/inject-cloud.mjs` (don't hand-add the tags).
- **Changed which scripts ship in the inject block?** Update `js/inject-cloud.mjs` then run `node js/update-inject.mjs`.
- **Skipped pages:** `/403.html`, `/404.html`, `/404-safe.html`, `/404-building.html`.

Backend API (in `chat/` repo) uses `?origin=jimmyqrg` on `/api/saves*`. See `DEVELOPERS.md` for full routes.

## Games

Two locations:
- **`q/g/`** (this repo) — 137+ game folders, each its own `index.html` with the inlined `__JqrgLoaderLoaded` IIFE in `<head>`.
- **`../jg/g/`** (sibling repo) — additional game packs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [indiamonda/indiamonda.github.io](https://github.com/indiamonda/indiamonda.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
