---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**kymostudio** — a diagram-as-code DSL that compiles declarative source into **animated SVG** (plus Figma / Excalidraw / WebP). Monorepo (layout mirrors Remotion): two publishable libraries under `packages/`, sharing one version number.

- `packages/python` (PyPI `kymostudio`, CLI `kymo`) — the Python implementation: DSL parser, BPMN importer, layout engine, and renderers.
- `packages/js` (npm `kymostudio`) — an **independent TypeScript implementation with equivalent functionality** (its own data model, icon library, DSL parser + layout + alignment resolver (`dsl.ts`/`layout.ts`/`alignment.ts`, exposed as `parse`/`parseDiagram`), BPMN importer, and SVG renderer `renderSVG`). **Not a port** of the Python package — the two are separate codebases developed in parallel and kept at feature parity.
- Shared root assets consumed by both: `icons/`, `samples/`, `docs/`. The websites live under `packages/`: `packages/website` (kymo.studio — React landing + client-side playground at `/app/`), `packages/docs` (docs.kymo.studio — RSPress / React + MDX, self-contained content under `packages/docs/docs/`), `packages/editor` (editor.kymo.studio — client-side editor). All three deploy to **Cloudflare Pages** (root `website/` is an empty legacy dir).

Python requires **>=3.13** and is managed with **uv**.

## Commands

```bash
# Python (run from packages/python)
uv run --group dev python -m pytest -q                          # all tests
uv run --group dev python -m pytest tests/test_dsl.py::test_x   # single test
uv run kymo ../../samples/aiq.kymo                           # .kymo -> .svg
uv run kymo <file> --animate | --figma | --excalidraw           # other targets
uv run kymo ../../samples/order.bpmn                            # .bpmn -> .svg (BPMN import)

# Regenerate golden SVGs after an INTENTIONAL renderer/layout change (see gotcha):
KYMO_UPDATE_GOLDEN=1 uv run --group dev python -m pytest tests/test_diagrams.py tests/test_layout.py tests/test_edges.py

# JavaScript (run from packages/js)
npm test                # builds TS to dist/ then `node --test`
npm run build           # tsc -> dist/ (JS + .d.ts)
npm run typecheck       # tsc --noEmit
npm run build-manifest  # regenerate the icon manifest from root icons/

# Website (run from packages/website) — landing + playground at /app/
./build.sh              # assemble dist/ from committed bundles (no JS recompile)
./build.sh --bundle     # also rebuild src/landing.bundle.js + app/kymo.bundle.js (esbuild)
npm run dev             # serve dist/ at http://localhost:4321

# Deploy a website to Cloudflare Pages manually (wrangler OAuth must be
# logged in; --branch=main = production). Local auth identity: CLAUDE.local.md.
npx wrangler pages deploy dist --project-name=kymo-studio --branch=main
```

CI (`.github/workflows/test.yml`) runs `pytest -q` (Python) and `npm test` (JS) per package.

**Deploys**: pushing to `main` auto-deploys via `deploy-website.yml` / `deploy-docs.yml` / `deploy-editor.yml` (Cloudflare Pages projects `kymo-studio` / `kymo-docs` / `kymo-editor`, path-filtered). The wrangler command above is the manual path — note a later `main` push re-deploys whatever is on `main`, so land the source change too or it gets overwritten.

**Local dev ports (fixed by convention).** The `kymo-mcp` worker's `ALLOWED_ORIGINS` CORS whitelist (`packages/mcp/src/index.ts`) only admits two localhost origins, so serve each site on its assigned port or its cross-origin calls to `api.kymo.studio` get blocked:
- **`icons.kymo.studio`** (`packages/website-icons`) → **8231** — e.g. `cd packages/website-icons && ./build.sh && (cd dist && python3 -m http.server 8231)`. Needs the API for the live brands/overlay catalogue + the admin panel.
- **`editor.kymo.studio`** (`packages/editor`) → **8099**.

Both ports (and their `127.0.0.1` forms) are whitelisted; any other port is rejected by the worker's `/api/*` CORS.

## Architecture (packages/python/src/kymo)

The renderer is deliberately **dumb**: `model.py` holds plain dataclasses (`Component`, `Region`, `Edge`, `Diagram`) and the emitters just turn that data into output. To change a diagram you change the data, never the renderer.

**Pipeline** (front-ends produce a `Diagram`, then a shared back-end resolves + renders):

1. **Source → `Diagram`**
   - `.kymo` (the DSL) → `dsl.py:parse()` — line-oriented grammar; parsing is purely declarative (collects elements, validates nothing, computes no positions).
   - `.bpmn` (BPMN 2.0 XML) → `from_bpmn.py:parse()` — reads the file's Diagram-Interchange geometry.
   - `.py` source → a module exposing `DIAGRAM` (+ optional `LAYOUT`, `EXTERNAL_LAYOUT`).
2. **`layout.py:layout()`** — only when a DSL `layout { … }` tree is present; positions members of auto-layout frames.
3. **`alignment.py:resolve_alignments()`** — the post-parse resolver (5 passes): auto-layouts, parent/child anchoring, region auto-bounds, fan-in / trunk-lane edge staggering, and auto-canvas sizing. This is where positions actually get computed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kymostudio/kymostudio](https://github.com/kymostudio/kymostudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
