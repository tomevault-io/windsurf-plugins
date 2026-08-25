---
trigger: always_on
description: Root guide for any AI agent (Claude Code, opencode, Copilot, Cursor, …). It covers the whole monorepo: what lives
---

# Trazor — Agent Instructions

Root guide for any AI agent (Claude Code, opencode, Copilot, Cursor, …). It covers the whole monorepo: what lives
where, how to run and verify things, and the conventions that apply everywhere.

**Area guides — read the one covering the directory you are editing, in addition to this file:**

| Editing…                                               | Read first                                             |
| ------------------------------------------------------ | ------------------------------------------------------ |
| `packages/trace/` — the tracer, the flagship algorithm | [`packages/trace/AGENTS.md`](packages/trace/AGENTS.md) |
| `apps/web/` — the Vue 3 studio UI                      | [`apps/web/AGENTS.md`](apps/web/AGENTS.md)             |

Reference material worth opening when you need the map rather than the rules: [`ARCHITECTURE.md`](ARCHITECTURE.md)
(whole repo), [`packages/trace/ARCHITECTURE.md`](packages/trace/ARCHITECTURE.md) (the tracer),
[`docs/CONTRACTS.md`](docs/CONTRACTS.md) (exact package APIs), [`docs/REFERENCES.md`](docs/REFERENCES.md)
(algorithm & model sources), [`docs/ML_STRATEGY.md`](docs/ML_STRATEGY.md) (ML strategy: where ML fits, how determinism is
scoped for WebGPU, and how to build a training set), and [`docs/ML_ROADMAP.md`](docs/ML_ROADMAP.md) (the prioritized ML &
vectorization plan).

## Project overview

A **fully client-side** raster → SVG vectorization studio. It decodes an image in the browser, traces it to clean,
editable, cuttable SVG, and never sends anything to a server — no upload, no account, no backend. The output is a
static site deployable to GitHub Pages or any static host.

The **tracing algorithm is the product**: a Potrace-class curve chain (Selinger 2003, clean-room — no GPL code)
applied per color layer, plus a shared boundary graph for seam-free cutout partitions and skeleton-based centerline
tracing. On top of it sit target profiles (illustration, logo, vinyl cut, laser, pen plotter, stencil, …), data-derived
palette suggestions, and optional on-device ML (background removal, click-to-segment). See [`README.md`](README.md) for
the feature tour and [`docs/REFERENCES.md`](docs/REFERENCES.md) for the literature behind each stage.

## Repository layout

```
packages/                  Algorithm packages, consumed by name (@trazor/*). Pure TS, no DOM (except ml).
  core/                    @trazor/core — shared types, settings schema + profiles, Oklab color math,
                           geometry, path model, deterministic PRNG. Zero deps. Everything depends on it.
  raster/                  @trazor/raster — resize, denoise, background flatten, k-means++ quantization,
                           Otsu/adaptive threshold, morphology, Zhang-Suen thinning, chamfer distance.
  trace/                   @trazor/trace — THE tracer: crack decomposition, Potrace chain, shared boundary
                           graph (seam-free cutout), centerline extraction, Schneider fitting.
  svg/                     @trazor/svg — compact SVG serialization + output analysis.
  engine/                  @trazor/engine — mode pipelines, progress/cancellation, warnings, worker + client.
  ml/                      @trazor/ml — background removal & click-to-segment, plus the learned edge,
                           cleanup & signed-field conditioning models, via onnxruntime-web. Browser-only.
  assist/                  @trazor/assist — image statistics → recommended settings & suggested palettes.
apps/
  web/                     @trazor/web — Vue 3 + Pinia + Vite studio UI. The deployable app.
docs/                      CONTRACTS.md (package APIs), REFERENCES.md (sources), screenshot.png.
scripts/                   e2e.mjs — real-browser smoke test / screenshot generator.
shared configs             .oxlintrc.json, .oxfmtrc.json, tsconfig.base.json, tsconfig.packages.json, vitest.config.ts.
```

**Where a new workspace goes** — keep the split consistent:

- `packages/*` — an algorithm package consumed _by name_ (`@trazor/*`), pure and testable in Node. No DOM APIs
  except `@trazor/ml` (which guards all browser access behind functions so it still imports in Node).
- `apps/*` — a deployable surface. Today just `web`.

Every workspace is listed in the root [`package.json`](package.json) `workspaces` array (`packages/*`, `apps/*`).
Packages resolve each other through the workspace symlink and export TypeScript source directly (`"exports": "./src/index.ts"`) —
Vite and Vitest consume the source, so there is **no per-package build step**; only `apps/web` builds (via Vite).

## Quick start

Prerequisites: **Node.js 22+**, npm, Git. All commands run from the **repo root**.

```bash
npm install
npm run dev          # Vite dev server → http://localhost:5173
```

Everything runs in the browser; there is no database, server or configuration.

## Commands

From the repo root:

| Command                     | Purpose                                                                                                                                                             |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PhenX/Trazor](https://github.com/PhenX/Trazor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
