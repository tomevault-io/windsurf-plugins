---
trigger: always_on
description: This repo is a Next.js catalog for open-source CAD parts. The source of truth is STEP files plus human-authored catalog metadata; GLB previews, PNG thumbnails, and the SQLite catalog are generated artifacts. Generated previews are published to Vercel Blob rather than committed.
---

# Agent Guide

This repo is a Next.js catalog for open-source CAD parts. The source of truth is STEP files plus human-authored catalog metadata; GLB previews, PNG thumbnails, and the SQLite catalog are generated artifacts. Generated previews are published to Vercel Blob rather than committed.

## Quick Commands

- Dev server: `npm run dev`
- Catalog SQLite metadata only: `node scripts/generate-catalog.mjs`
- Catalog build: `npm run catalog:build`
- Preview asset sync: `npm run catalog:sync-assets`
- Non-mutating catalog validation: `npm run catalog:check`
- Lint: `npm run lint`
- TypeScript check: `npx tsc --noEmit`
- Full app check: `npm run check`

`npm run catalog:build` rebuilds local GLB/PNG preview artifacts only; it does not update `catalog/parts.sqlite`. Preview Blob paths are derived from each part's STEP SHA-256, so SQLite is reproducible from source catalog metadata plus STEP files. Tune paired export lanes with `STEP_PARTS_EXPORT_CONCURRENCY`; it defaults to 2. Production `npm run catalog:sync-assets` uploads only missing deterministic Blob assets and generates local GLB/PNG files only for those missing parts.

For metadata-only changes to `catalog/parts.json`, do not run `npm run catalog:build`. Run `node scripts/generate-catalog.mjs` instead to refresh `catalog/parts.sqlite` without rebuilding GLB/PNG artifacts. Only run `catalog:build` when STEP files or preview assets need to be added, refreshed, or repaired.

The thumbnail exporter starts a local render server on `127.0.0.1`; sandboxed runs may need approval/escalation.

## Repo Shape

- `catalog/parts.json`: human-authored part records only.
- `catalog/parts.sqlite`: generated catalog metadata used by the app.
- `catalog/taxonomy.json`: narrow guardrails for rigid, repeatable families only.
- `catalog/step/{id}.step`: canonical STEP assets.
- `public/glb/{id}.glb`: local generated interactive previews, ignored by Git and synced to Vercel Blob.
- `public/png/{id}.png`: local generated 512x512 thumbnails, ignored by Git and synced to Vercel Blob.
- `scripts/add-part.mjs`: single-part helper for local STEP files.
- `scripts/generate-catalog.mjs`: regenerates catalog SQLite metadata only.
- `scripts/export-assets.mjs`: converts STEP to GLB and renders PNGs.
- `scripts/check-catalog.mjs`: validates source schema, taxonomy guardrails, STEP sanity, generated SQLite rows, and optionally published Blob assets.
- `src/components/part-directory.tsx`: searchable catalog UI and card previews.
- `src/components/part-viewer.tsx`: detail-page 3D viewer with PNG fallback.
- `src/lib/part-query.ts`: server-side search/filter/sort behavior.
- `src/app/v1/*`: catalog API routes.

STEP/STP files in `catalog/step` are Git LFS assets via `.gitattributes`. GLB/PNG previews are Vercel Blob assets and should not be committed.

Local development and catalog validation serve STEP files from `catalog/step` through `/step/{id}.step`, so local downloads match the SQLite byte sizes and hashes. Production API `stepUrl` values and single-download redirects use commit-pinned GitHub LFS media URLs by default; set `STEP_PARTS_GITHUB_REF`, `STEP_PARTS_GITHUB_REPOSITORY`, or `STEP_PARTS_GITHUB_OWNER` plus `STEP_PARTS_GITHUB_REPO` to override the GitHub target.

## Adding Parts

Prefer `npm run catalog:add` when adding a single local STEP file. It copies the STEP into `catalog/step`, appends `catalog/parts.json`, rebuilds generated assets, and validates the catalog.

For scripted use:

```bash
npm run catalog:add -- \
  --step /absolute/path/to/part.step \
  --id stable_snake_case_id \
  --name "Human part name" \
  --description "One searchable sentence." \
  --category electronics \
  --family raspberry-pi \
  --tag board \
  --tag microcontroller \
  --alias "Useful alias" \
  --attr manufacturer="Example"
```

Use `--dry-run` first when metadata is uncertain.

For large target lists, use a file instead of a long shell argument:

```bash
find catalog/step -name '*.step' > /tmp/changed-steps.txt
npm run catalog:build -- --targets-file /tmp/changed-steps.txt
npm run catalog:build -- --targets @/tmp/changed-steps.txt
```

Target list files accept one part id or asset path per line. Blank lines and lines beginning with `#` are ignored. Entries can be part ids, bare filenames, absolute or relative paths, or `catalog/step/{id}.step` paths.

For large catalog additions, use `catalog/taxonomy.json` as a lightweight context file, not as a complete ontology. It is only for rigid families with predictable identity fields and required attributes, such as standardized screws, washers, bearings, stock profiles, and helper geometry. Use it to choose existing rigid families and to detect likely duplicates. For flexible brand, product, electronics, actuator, or one-off families, inspect nearby examples in `catalog/parts.json` or query `catalog/parts.sqlite`; do not add taxonomy entries just to make every family listed.

### Metadata Rules

Keep source records limited to these keys:

- `id`: stable snake_case ASCII id matching asset filenames.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [earthtojake/step.parts](https://github.com/earthtojake/step.parts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
