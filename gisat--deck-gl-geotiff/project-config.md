---
trigger: always_on
description: - Install dependencies (root workspace): `yarn install`
---

# Copilot Instructions for `deck.gl-geotiff`

## Build, lint, and verification commands

- Install dependencies (root workspace): `yarn install`
- Build library (root): `yarn build`
- Build library directly (workspace): `yarn workspace @gisatcz/deckgl-geolib rollup`
- Lint library sources: `yarn lint`
- Lint with fixes: `yarn lintFix`
- Run example app locally: `yarn example` (or `yarn start` to build library first and then run example)
- Build example app: `yarn workspace example build`
- Type-check example app: `yarn workspace example typecheck`

### Test command note

There is no unit/integration test runner configured in this repo right now (no Jest/Vitest/Mocha scripts). Validation is done through linting, building, and exercising behavior in the `example` app.

## High-level architecture

This repository is a Yarn workspace monorepo with:

- `geoimage/`: published library package (`@gisatcz/deckgl-geolib`)
- `example/`: Vite + React app for manual verification and demos

Core flow for rendering tiles:

1. Deck.gl layer (`CogBitmapLayer` or `CogTerrainLayer`) owns/uses a `CogTiles` instance.
2. `CogTiles` loads COG metadata using `geotiff`, computes zoom/resolution lookup tables, and fetches tile rasters with edge padding logic.
3. `CogTiles` delegates raster conversion to `GeoImage.getMap(...)`.
4. `GeoImage` is a facade that routes to:
    - `BitmapGenerator.generate(...)` for 2D `ImageBitmap` output
    - `TerrainGenerator.generate(...)` for 3D mesh output
5. Deck.gl sublayers render the generated bitmap/mesh data.

Important design split:

- Rendering logic is intentionally separated into `BitmapGenerator` and `TerrainGenerator`.
- `GeoImage` should stay orchestration-focused, not absorb low-level pixel/mesh algorithms.

## Key project conventions

- COG assumptions are Web-Mercator-oriented. Inputs are expected to be web-optimized/tiled GeoTIFFs (typically 256 tile size, EPSG:3857 workflows).
- Terrain tiles use `tileSize + 1` (257) when needed to support seam stitching and skirt handling.
- Channel selection supports both 1-based (`useChannel`) and 0-based (`useChannelIndex`) options; `useChannelIndex` is derived when omitted.
- Core options are merged against `DefaultGeoImageOptions`; preserve this merge behavior when adding new options.
- For bitmap performance, keep the LUT optimization path in `BitmapGenerator` for 8-bit data.
- Library output is dual-format (ESM + CJS) via Rollup; keep external dependency handling aligned with `geoimage/rollup.config.mjs`.
- Workspace-level scripts are the source of truth; prefer running commands from repo root using `yarn workspace ...`.

## PR Description Structure

When creating a pull request, use the following structure for the PR description (adapted from PR #137):

---
# PR Title (see workflow rules)

## Summary
Briefly describe the purpose and scope of the PR, referencing the baseline (e.g., previous release or PR).

## Major Features / Changes
1. **Feature/Change 1**: Short description and impact
2. **Feature/Change 2**: ...

## Security & Maintenance
- List any dependency/security updates, infra changes, or maintenance tasks

## Changes Summary
- Number of files changed, insertions/deletions, and key PRs merged (if relevant)

## Key Files Modified
- List of most important files or directories changed

## Testing
- How the changes were validated (lint, build, manual, etc.)

## Notes
- Any extra context, caveats, or follow-up actions
---

## Existing repository-specific workflow rules

- All PR titles and commit messages must follow the rules enforced by repository workflows:
    - **PR titles** must use the [Conventional Commits](https://www.conventionalcommits.org/) format (e.g., `feat: ...`, `fix: ...`, etc.), except for `dev → master` merges, which use `Merge \`dev\` → \`master\``.
    - **Commit messages** (especially for release and automation) must also follow the Conventional Commits standard. Automated commits (e.g., lint fixes) use `style: fix lint errors`.
    - These rules are enforced by GitHub Actions (semantic PR title validation, semantic-release, etc.).
- Follow step-by-step execution for implementation/review tasks: one checklist item at a time, explain what changed, then wait for explicit user confirmation to continue.
- Use hierarchical numbering (`1.1`, `1.2`, ...) in plans/checklists.
- Keep plan/instruction artifacts in `.plan/` with `YYYY-MM-DD-kebab-case.md` naming.
- **Always ask for explicit user confirmation before running `git commit`.** Never commit autonomously.
- Before finalizing substantial work, prepare `PR_DESCRIPTION.md` using:
    - Base branch logic: if current branch is `dev`, use `master` as base; for all other branches, use `dev` as base.
    - `git diff <base_branch> --stat` and `git log <base_branch>..HEAD`
    - `PR_DESCRIPTION.md` is a **temporary working file** — never stage or commit it.
    - PR title format `Merge \`branch\` → \`target\`` is reserved for `dev → master` merges only. Feature branches use descriptive titles (e.g. `feat: ...`).

## Public API

Exports from `@gisatcz/deckgl-geolib`:
```ts
import { CogBitmapLayer, CogTerrainLayer, CogTiles, GeoImage } from '@gisatcz/deckgl-geolib';
import type { GeoImageOptions } from '@gisatcz/deckgl-geolib';
```

### Layer props quick-reference


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gisat/deck.gl-geotiff](https://github.com/Gisat/deck.gl-geotiff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
