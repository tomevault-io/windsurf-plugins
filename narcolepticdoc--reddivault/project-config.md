---
trigger: always_on
description: Persistent project memory for Claude Code sessions. Read at session start; update when
---

# RedditVault — Claude Code Context

Persistent project memory for Claude Code sessions. Read at session start; update when
significant decisions are made or features added. Keep it **current-state** — decision
history lives in `docs/transcripts/` (see `journal.txt` for the index), not here.

RedditVault is a personal Reddit saved-items manager built to work around Reddit's
~1,000-item API limit. A static PWA (IndexedDB-first) syncs with Supabase; saves are
captured via a Chrome extension, a bookmarklet, or the Reddit RSS feed.

**Live URL:** https://reddivault.vercel.app · **Current version:** v0.9.19.0

---

## Hard Rules — every change must respect these

1. **Zero-build, static deploy.** Native ES modules loaded directly by the browser —
   no bundler, no package.json, no transpiler, no build step. Third-party libs (Dexie,
   PapaParse) come from CDN `<script>` tags in `pwa/index.html`. Vercel serves the files
   as-is.
2. **Version bumps on every deployed change.** Bump `APP_VERSION` in `pwa/js/state.js`
   AND `VERSION` in `pwa/sw.js` (format `major.minor.patch.hotfix`). The SW precaches a
   hand-maintained `ASSETS` array keyed off `VERSION` — **a new `js/*.js` file must be
   added to `ASSETS`** or it won't work offline.
3. **The `window` bridge.** Rendered HTML uses ~140 inline `onclick="fn(…)"` handlers
   that resolve against global scope. `app.js` does `Object.assign(window, ...modules)`,
   so any function referenced by an inline handler must be **`export`ed from its module**
   — the bridge picks it up automatically, zero template changes.
4. **Rating three-state.** `rating` is `null` (unrated) | `0` (thumbs-down) | `1`–`5`
   (stars). Always propagate with `?? null`, **never `|| null`** (which coerces a real
   `0` to unrated).
5. **Soft delete only.** "Permanently deleted" items stay in the DB with
   `isPermanentlyDeleted: true` + `deletedAt` (Supabase `deleted_at`) so feed sync can't
   resurrect them. Actual removal is only the explicit Purge action.
6. **Single ingestion path.** `mapRedditChild(child, source)` is the only mapper from a
   raw Reddit listing child to an item — feed sync and the bookmarklet drain both use it.
   The bookmarklet writes **only** to the `reddit_inbox` staging table, never to
   `reddit_saves`; all real writes go through the app's dedup-aware drain.
7. **`localEditAt` pull guard.** Every per-item local mutation stamps
   `item.localEditAt` with an ISO timestamp (client clock). `syncFromSupabase` and
   `deltaPullBeforePush` must skip overwriting any item whose `localEditAt` is newer
   than the pull's start time (both on the client clock), so an in-flight pull can't
   clobber a fresh edit.
8. **iOS quirks.** `font-size: 16px` on all inputs (prevents zoom-on-focus); viewport
   pinch-zoom lock is user-controllable (`state.disableZoom`). The bookmarklet string
   must stay quote-free (no double quotes, no backticks, no inline `onclick`) — banners
   use the `done()` helper with `textContent`.

---

## File Structure

```
/
├── CLAUDE.md                          ← this file
├── SETUP.md                           ← end-user setup guide
├── supabase-schema.sql                ← DB schema (fresh install + migration sections)
├── vercel.json                        ← Vercel config (auto-deploys from GitHub)
├── pwa/
│   ├── index.html                     ← shell: markup + CDN libs + module entry
│   ├── styles.css                     ← all app CSS
│   ├── sw.js                          ← service worker (VERSION + ASSETS list)
│   ├── manifest.json, icon-*.png
│   └── js/                            ← the app, native ES modules
│       ├── app.js                     ← entry: imports all, window bridge, bootstrap
│       ├── state.js                   ← APP_VERSION, Dexie db + version chain, state, syncLog
│       ├── util.js                    ← escHtml, fmtDate, showToast, renderMarkdown,
│       │                                openLink, fullUrl, ratingDisplay, applyZoomSetting
│       ├── core.js                    ← init, loadConfig/loadData, rebuildTagCache,
│       │                                rebuildFilterLists, reconcileDirtyState, markDirty
│       ├── enrich.js                  ← Arctic Shift + Reddit enrichment, rate limiting
│       ├── cloud.js                   ← Supabase REST (supabaseFetch), push/pull/delta sync,
│       │                                preferences push/pull, retry/dirty machinery
│       ├── feed.js                    ← RSS/JSON feed sync, _buildProxyUrl, config save
│       ├── bookmarklet.js             ← buildInboxBookmarklet, drainInbox, ingestChildren,
│       │                                mapRedditChild, applyScoreUpdates,
│       │                                importPastedBookmarklet, cspProbeBookmarklet
│       ├── dataio.js                  ← CSV import, JSON backup/restore, repairs, delete
│       ├── search.js                  ← parseSearchQuery, itemMatchesTokens, _parseWildcard,
│       │                                affinityScore, sortItems, filteredItems,
│       │                                list-options helpers (serialise/applyListOptions)
│       ├── items.js                   ← item/list mutations (favourite, rate, trash, delete,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [narcolepticdoc/reddivault](https://github.com/narcolepticdoc/reddivault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
