---
trigger: always_on
description: A text-to-flowchart editor. Users type in a custom DSL and see a live graph visualization. Supports local (sandbox) and hosted (cloud) charts, with a pro subscription tier (~500 paying customers).
---

# Flowchart Fun

A text-to-flowchart editor. Users type in a custom DSL and see a live graph visualization. Supports local (sandbox) and hosted (cloud) charts, with a pro subscription tier (~500 paying customers).

**Live site:** flowchart.fun (dev site: dev.flowchart.fun)
**GitHub:** tone-row/flowchart-fun
**Single maintainer:** Rob Gordon (~1400+ commits)

## Monorepo Structure

**pnpm workspaces** monorepo with 4 packages:

```
flowchart-fun/
├── app/        → React frontend (CRA / react-scripts 5.0.1)
├── api/        → Vercel serverless functions (Node.js)
├── shared/     → Shared TypeScript types & constants (compiled with tsc)
├── formulaic/  → React form-building library (compiled with microbundle)
```

**Package manager:** pnpm (v10)
**Node version:** 22 (see `.node-version`)

> **Maintainer note:** The monorepo structure with all these packages partly exists because of CRA/webpack limitations that needed workarounds. Some of this sprawl is historical.

## Getting Started

```bash
pnpm install

# IMPORTANT: shared and formulaic must be built before the app can run
pnpm -F shared build
pnpm -F formulaic build

# Start the dev server (use this for local development)
pnpm start
```

### Dev Server: `pnpm start` vs `pnpm dev`

**Use `pnpm start`** (runs `vercel dev`) for local development. This starts the React app AND exposes the `/api/*` serverless functions. Required for features like feedback submission, AI prompts, auth flows, and Stripe — and required for E2E tests to pass.

`pnpm dev` (runs `pnpm -F '*' --parallel dev`) only starts the React app + workspace watchers without API routes. Use this only if you're working on purely frontend changes and don't need API access.

Both serve on port 3000 by default.

### Environment Variables

The app requires env vars in `app/.env.local`. Pull from Vercel with:
```bash
pnpm env:pull    # runs: vercel env pull app/.env
```

Key variables (see `app/.env.example`):
- `REACT_APP_SUPABASE_URL` / `REACT_APP_SUPABASE_ANON_KEY` — Supabase auth & DB
- `REACT_APP_STRIPE_KEY` / `STRIPE_KEY` — Stripe payments
- `REACT_APP_SENTRY_ENVIRONMENT` — Sentry error tracking
- `SENDGRID_API_KEY` — Email via SendGrid
- `NOTION_ACCESS_TOKEN` — Notion integration (blog/roadmap)

## Build Order

Strict dependency order:

```
formulaic → shared → app
```

Root `build` script: `pnpm -F formulaic build && pnpm -F shared build && pnpm -F app build`

If you modify `shared/` or `formulaic/`, rebuild them before the app will pick up changes (or use `pnpm dev` which runs watch mode for all).

## Release Workflow

1. Create feature branch off `dev` (e.g., `robgordon/ff-485-add-folders`), merge back into `dev` via PR
2. On `dev`, bump version in `app/package.json` and commit as `chore: version {type}` (where type is `feature`, `fix`, `patch`, or `hotfix`)
3. Create PR from `dev` → `main` with title being just the version number (e.g., `v1.63.1`)
4. Merge triggers `.github/workflows/release.yml` which uses `justincy/github-action-npm-release` to auto-create a GitHub Release + tag from `app/package.json` version
5. Vercel auto-deploys from `main`

Multiple feature PRs may accumulate on `dev` before a single version bump + release to `main`, or a single feature may get its own release. All merges to `main` come exclusively from `dev` — never direct feature branch → main.

The version in `app/package.json` is the source of truth. Tags are created automatically.

## Core Architecture

### The Graph-Selector DSL

The core feature is a **custom text DSL** parsed by the `graph-selector` npm package (v0.13.0). This is the most fundamental dependency in the project — it's what makes flowchart.fun work.

Key pipeline: **Text → parse() → toCytoscapeElements() → Cytoscape renders**

Main parsing entry point: `app/src/lib/getElements.ts`

> **Maintainer note:** `graph-selector` lives in a separate repository (also maintained by Rob). It has extensive unit tests. The workflow for parser changes is: make changes in that repo, publish to npm, bump version here. It was made separate thinking others might use it, but in practice nobody else does. Moving it into this monorepo is an option if it would reduce friction — the tests would come with it.

### Document Format

Charts are stored as text with embedded metadata:

```
Node A
  Node B
    Node C

=====
{"themeEditor": {...}, "cytoscapeStyle": "..."}
=====
```

- `=====` — Current delimiter for JSON metadata (`newDelimiters` in `app/src/lib/constants.ts`)
- `~~~` — Legacy delimiter (YAML frontmatter via gray-matter)
- `¼▓╬` — Legacy hidden graph options divider

The parsing logic in `app/src/lib/prepareChart/prepareChart.ts` handles all three formats and merges them, with migration logic for old layouts.

> **DO NOT REMOVE** the legacy delimiter handling. There are ~500 paying customers with existing charts that may use older formats. Removing backward compatibility could break their projects.

### State Management

**Primary:** Zustand stores

| Store | File | Purpose |
|-------|------|---------|
| `useDoc` | `lib/useDoc.ts` | Document text, metadata, and chart details |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tone-row/flowchart-fun](https://github.com/tone-row/flowchart-fun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
