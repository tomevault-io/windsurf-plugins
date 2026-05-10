---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Web GIS for visualizing iTIC Thailand vehicle probe data (~1.86M points/day). Two pieces:

- `webgis/preprocess/preprocess.mjs` — Node CLI. CSV → 20 B/record packed binary.
- `webgis/app/` — Vite + deck.gl + MapLibre frontend reading those binaries as typed arrays.

Top-level `run.cmd` / `run.sh` orchestrate install → preprocess → dev-server. README is **user-facing only**; anything dev-side belongs here.

## Commands

```bash
.\run.cmd                # install + preprocess (if needed) + dev server
.\run.cmd --serve        # skip preprocess
.\run.cmd --rebuild      # force re-preprocess
.\run.cmd --limit 1      # only first DEFAULT_DATES entry

# preprocess flavors
cd webgis/preprocess
DATES=20250101,20250201 node preprocess.mjs
APPEND=1 DATES=20250919 node preprocess.mjs   # add one day, keep existing vids
KEEP_TMP=1 node preprocess.mjs                # leave .tmp/ for iterative re-runs

# frontend
cd webgis/app
npm run dev    # vite dev server
npm run build  # production build (also fast type/import sanity check)
```

No test suite. Smoke checks live under `webgis/tmp/` — write named `*.mjs`/`*.sh` rather than long inline `node -e "…"` to avoid permission-prompt thrash.

## Architecture

```
.csv.out → preprocess.mjs → YYYYMMDD.bin → binary.js → layers.js → deck.gl
                            (~36–44 MB)   (typed views) (binary attrs)
```

**Binary layout is a load-bearing contract.** Canonical definition is the header comment at the top of `webgis/preprocess/preprocess.mjs` — read it when touching the format. The frontend (`webgis/app/src/binary.js`, `layers.js`) maps the records section to **strided typed-array views** fed directly to deck.gl as `{value, size, stride, offset}` binary attributes; no per-record JS objects are ever allocated. Changing record size, offsets, or header layout means updating `HEADER_SIZE`/`RECORD_SIZE`/hard-coded offsets in `binary.js` **and** stride values in `layers.js`. Mismatches compile fine and silently render garbage.

**GPU filtering** via `DataFilterExtension` packs `[time_offset_sec, passFlag]` into `Float32Array(count*2)`. Slider drags only update `filterRange` — CPU never iterates per-record. Filter changes (gps/moving/speedMax/colorBy) rebuild the filterValues / colors buffers in one pass.

**Aggregation layers (Heatmap, Hexagon, polygon ROI) take a CPU path** — they can't use the GPU filter, so they extract a flat `Float32Array` of visible `[lon,lat,...]` once per render. Hexagon sub-samples to 200K.

**State is a plain object** in `webgis/app/src/main.js`. UI mutates it and calls `render()`, which rebuilds layers; deck.gl diffs them.

## Preprocess gotchas

- **Extraction backend.** On Windows the bundled `tar.exe`'s bzip2 is ~25× slower than 7-Zip (1.4 GB archive: ~25 min vs ~1 min). The script auto-detects `7z.exe` at standard install paths (or `SEVENZIP_EXE` env) and uses two-pass extraction (bz2→tar, then untar). POSIX uses `tar -xjf` directly (fast). Don't go back to `tar -xjOf | node` streaming — Node readline backpressures bzip2 against itself.
- **Tmp dir lifecycle.** `webgis/preprocess/.tmp/PROBE-YYYYMM/` created per run, deleted in `finally`. All dates sharing an archive are batched. `KEEP_TMP=1` preserves.
- **APPEND=1.** Seeds `vehicleDict` from existing `vehicles.json` and keeps non-reprocessed days from `meta.json`. Without it, a clean rebuild remaps vehicle indices and **silently invalidates every previously-shipped `.bin`**. Always APPEND when adding days to a published bundle.
- **Stale-data filter.** CSVs contain cached probes from prior years and `(0,0)` coords. Records outside `[fileDate-2d, fileDate+3d]` GMT+7, year<2000, or `(0,0)` are dropped. Don't relax — earlier prototypes had garbage bbox/t_min.
- **GMT+7 ↔ UTC.** Source timestamps are GMT+7; preprocess stores **UTC unix seconds** (`tUtc/1000 - 7*3600`); frontend `formatBkk()` re-adds 7h for display. Don't double-adjust.

## Frontend gotchas

- **`THAILAND_BBOX` in `main.js` overrides per-day bbox for initial map fit.** Bin-header bbox can be polluted by stale records leaking near `(0,0)`; the country-fixed extent keeps the first frame sane. Stats panel still shows the actual data bbox for diagnostics.
- **HeatmapLayer / HexagonLayer silently fail on software-rendered Chromium** (`Binding weightsTexture not set`) — affects headless smoke tests under `chrome-headless-shell`. Use real Chromium for those layers. Points layer is unaffected. Don't try downgrading deck.gl/luma.gl to "fix" — luma.gl 9.1↔9.3 export changes mean a clean pin needs `overrides` on every `@luma.gl/*`.

## Deploy (GitHub Pages shell + Cloudflare R2 data)

Two workflows on push to `main`:
- `.github/workflows/pages.yml` — builds `webgis/app/` and deploys `dist/` to Pages. **Strips `public/data/{*.bin,meta.json,vehicles.json}` before `npm run build`** so the artifact is just the JS/HTML/CSS shell (~few hundred KB).
- `.github/workflows/r2-sync.yml` — on changes under `webgis/app/public/data/**`, uploads all `.bin` + `meta.json` + `vehicles.json` to a Cloudflare R2 bucket via `wrangler r2 object put --remote`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [toruseo/iTIC-probe-viewer](https://github.com/toruseo/iTIC-probe-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
