---
trigger: always_on
description: Guidance for working in this repo. See `README.md` for the user-facing overview.
---

# CLAUDE.md

Guidance for working in this repo. See `README.md` for the user-facing overview.

## What this is

A static, single-page item/NPC/dungeon database for Tortoise-WoW (a 1.12 MaNGOS
fork), hosted on GitHub Pages at https://xian55.github.io/tortoise-db-viewer/.
There is **no backend**: the whole SQLite DB is shipped and queried in the
browser with the official `@sqlite.org/sqlite-wasm` build.

## Architecture (how it fits together)

```
../tortoise-wow/sql/base/*.sql        server MaNGOS SQL dumps (base world data)
../tortoise-wow/sql/database_updates/  incremental world migrations (patch content:
   *.sql                               new zones/NPCs/objects/quests). mangosd
                                       applies these at runtime; the build does too.
        │  scripts/build-db.mjs        stage raw tables -> apply migrations (in
        │                              timestamp order) -> normalize + index +
        │                              resolve chances
        ▼
public/data/tortoise.sqlite           one indexed DB (~34 MB), fetched whole
        │  src/db.js + src/db-worker.js (sqlite-wasm in a Worker, OPFS cache)
        ▼
src/queries.js  → src/table.js / src/render.js / src/hovercard.js / src/browse.js

F:/Game/Turtle WoW/Data/*.mpq         client patch MPQs (Turtle custom content)
        │  scripts/extract-icons.py    LOCAL ONLY — needs the client + StormLib
        ▼
assets/icons/custom/*.webp            committed source: 1 icon/file (extracted once)
scripts/data/item-display-supplement.json  committed: display_id -> icon, for every
                                      item row missing/stale in the server SQL dump
        │  scripts/build-atlas.py      pack icons into one sprite sheet
        ▼
public/icons/custom-atlas.{webp,json} the shippable atlas (render.js draws sprites)
```

- **Whole-DB load, not HTTP range.** GitHub Pages gzips responses (including 206
  partials) with `Content-Range` reporting the *compressed* size, which corrupts
  byte-range reads — so sql.js-httpvfs is unusable here. We download the whole
  file once (gzip is transparent for a full GET). SQLite runs in a **Web Worker**
  (`src/db-worker.js`); `src/db.js` is a thin message client. The worker is
  required for the durable **OPFS** cache: the SAHPool VFS's
  `FileSystemSyncAccessHandle` is **Worker-only** (it's `undefined` on the main
  thread in Chrome), so the old main-thread SAHPool always failed and re-fetched
  the ~58 MB DB every visit. In the worker OPFS persists (no COOP/COEP needed,
  SAHPool not the Atomics VFS); falls back to an in-memory deserialize when OPFS
  is unavailable. Trade-off: query results cross the worker boundary (structured
  clone) — negligible even for the big zone queries.
- **Cache invalidation:** `build-db.mjs` writes `data/version.json` with a
  content hash. `db.js` keys the download URL (`?v=`) and the OPFS filename by
  that hash and wipes old copies, so a new deploy auto-refreshes clients.
- **Routing** is query-param based (SPA, no server rewrites): `?item=`, `?npc=`,
  `?quest=`, `?faction=`, `?zone=`, `?subzone=`, `?dungeon=`, `?dungeons`,
  `?browse=items|npcs|quests|factions|zones|subzones|crafting`, `?search=`, `?compare=a:b:c`
  (item comparison), `?talents=<class>` (talent calculator), `?random`. `route()`
  checks `?browse=` (and `?compare=`) **before** the singular entity params (browse
  URLs carry filter params like `faction=a` that collide otherwise). See `src/main.js`.
  The **dataset** (main vs dev DB) is orthogonal to `route()` — it's chosen from the
  path (`/dev/…`) in `src/config.js`, not a `route()` branch (see "Two datasets").
- **Item browse gear features** (`src/browse.js`): the multi-criteria stat filter
  (`stats=key,op,val|…`, `match=all|any` for AND/OR) and **stat-weight ranking**
  (`weights=key:w|…` + `STAT_WEIGHT_PRESETS`) add a computed, sortable **Score**
  column — both resolve stats through the derived `item_stats` table. Selecting rows
  → **Compare** builds a `?compare=` URL; a localStorage compare tray (main.js
  `renderCompareTray`) collects items across pages. The selection bar itself is
  `src/selbar.js`, shared with the search results' Items tab.
- **Zone maps use Leaflet + a Pixi GPU overlay** (`L.CRS.Simple`,
  `leaflet-pixi-overlay` + `pixi.js`, all npm, lazy-loaded as one chunk via
  `src/zonemap.js`). A zone page renders the in-game parchment image
  (`public/maps/<areaId>.webp`) and plots spawn markers; world (x,y) → image px
  via the zone's WorldMapArea bounds (`lat=H*(x-locbottom)/(loctop-locbottom)`,
  `lng=W*(locleft-y)/(locleft-locright)`). Markers are **Pixi sprites** in one
  `PIXI.Container` (a tinted disc texture for category dots, atlas/CDN textures
  for focus/object icons) so huge zones (~12k spawns) pan/zoom on the GPU.
  Category toggles are tiny `L.Layer`s flipping `sprite.visible`; hover tooltip +
  click-nav use a throttled nearest-visible-sprite hit-test (no per-marker DOM).
  The previous overlay is `destroy()`ed on re-init to free its WebGL context.
- **Search is unified + FTS-backed.** `?search=` renders a tabbed page across
  items/NPCs/quests/dungeons/zones/subzones; the top-bar input also shows a live flat

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Xian55/tortoise-db-viewer](https://github.com/Xian55/tortoise-db-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
