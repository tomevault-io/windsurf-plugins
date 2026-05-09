---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**Watchboard** — a multi-topic intelligence dashboard platform. Each "tracker" is a self-contained dashboard with its own data, sections, map region, 3D globe, and AI update prompts. Built with Astro 5, TypeScript, and React islands. Data stored as JSON files per tracker, auto-updated via Claude Code Action (Max subscription OAuth).

Active trackers: **Iran Conflict**, **September 11**, **Chernobyl**, **Fukushima**, **Ayotzinapa**, **MH17 Shootdown**, **Mencho/CJNG**, **Culiacanazo**, and more. New trackers can be created in ~25 min via the `init-tracker.yml` GitHub Actions workflow.

## Commands

```bash
npm run dev          # Start dev server
npm run build        # Type-check + build static site to dist/ (postbuild runs pagefind indexing)
npm run preview      # Preview built site
npm run update-data  # Run AI data update for all trackers (requires ANTHROPIC_API_KEY or OPENAI_API_KEY)
TRACKER_SLUG=iran-conflict npm run update-data  # Update a single tracker
npm run backfill-media              # Enrich all events with og:image from source URLs
npm run backfill-media -- --dry-run # Preview what would change without writing
npm run backfill-media -- --tracker iran-conflict  # Single tracker only
```

## Deployment & Workflows

- **Build + deploy**: `.github/workflows/deploy.yml` — triggers on push to `main`, builds Astro, deploys `dist/` to GitHub Pages
- **Nightly data update**: `.github/workflows/update-data.yml` — runs at **14:00 UTC daily**, 3-phase pipeline:
  1. **Resolve** — finds eligible trackers, generates sibling brief (cross-tracker context) + review manifests (event gap detection)
  2. **Update** (matrix) — parallel jobs, 1 per tracker (max 5 concurrent), each with 50-turn budget via `claude-code-action`
  3. **Finalize** — downloads artifacts, validates JSON + Zod, runs fix agent if validation fails (1 retry), build gate (`npm run build`), commits data, collects + commits ingestion metrics
- **Init new tracker**: `.github/workflows/init-tracker.yml` — manual dispatch with slug, topic, start_date, region. Claude Code generates `tracker.json` + empty data files. Auto-chains into seed job.
- **Seed tracker data**: `.github/workflows/seed-tracker.yml` — manual dispatch for comprehensive historical backfill. Claude Code does deep web research and populates all sections.
- All data workflows use `claude-code-action` with `CLAUDE_CODE_OAUTH_TOKEN` (Max subscription) — no per-token API costs
- Each workflow produces a `$GITHUB_STEP_SUMMARY` with data inventory tables
- Legacy: `scripts/update-data.ts` still works with direct API keys (`ANTHROPIC_API_KEY` / `OPENAI_API_KEY`)

## Architecture

```
trackers/{slug}/
  tracker.json           →  src/pages/[tracker]/index.astro  →  Astro components
  data/*.json                (getStaticPaths + loadTrackerData)   (static .astro + React islands)
src/lib/tracker-config.ts  →  src/lib/tracker-registry.ts
  (Zod config schema)         (discovers all trackers at build time)
src/lib/schemas.ts         →  scripts/update-data.ts
  (data Zod schemas)           (AI nightly updater — iterates trackers)
```

### Tracker System

Each tracker is a directory under `trackers/` containing:
- `tracker.json` — config (slug, name, sections, map bounds/categories, globe presets, nav, AI prompts, political avatars, `updateIntervalDays`, `backfillTargets`, `ai.relatedTrackers`)
- `data/` — JSON data files (kpis, timeline, map-points, map-lines, etc.)
- `data/events/` — partitioned daily event files (`YYYY-MM-DD.json`)

Key files:
- `src/lib/tracker-config.ts` — `TrackerConfigSchema` (Zod) + types (`TrackerConfig`, `MapCategory`, `CameraPreset`, `NavSection`, `Tab`)
- `src/lib/tracker-registry.ts` — `loadAllTrackers()`, `loadTrackerConfig(slug)`, `getTrackerSlugs()`

### Data Layer (`trackers/{slug}/data/`)

Each tracker has its own data files: `kpis.json`, `timeline.json`, `map-points.json`, `map-lines.json`, `strike-targets.json`, `retaliation.json`, `assets.json`, `casualties.json`, `econ.json`, `claims.json`, `political.json`, `meta.json`, `digests.json`. The nightly update script modifies these files directly.

`update-log.json` tracks the last run time and per-section status.

`digests.json` stores RSS digest entries (date, title, summary, sectionsUpdated). The nightly updater prepends a new entry after each data update. These feed the site's RSS endpoints.

Data loader: `src/lib/data.ts` — `loadTrackerData(slug, eraLabel?)` uses `import.meta.glob` to load all tracker data at build time, validates via Zod, merges partitioned events. Cross-field validation: strike/retaliation map-lines must have `weaponType` + `time`.

### Event Media

Events can include a `media` array of `MediaItemSchema` objects: `{ type: "image"|"video"|"article", url, caption?, source?, thumbnail? }`. Media is displayed across multiple surfaces:

**Data supply:**
- Nightly pipeline (STEP 3.5 in `update-data.yml`) instructs Claude to populate `media` on significant events with article URLs and og:image thumbnails from Tier 1-2 sources

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArtemioPadilla/watchboard](https://github.com/ArtemioPadilla/watchboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
