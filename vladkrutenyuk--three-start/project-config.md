---
trigger: always_on
description: There are only **three commands** the maintainer ever needs to run by hand. Everything else (`prebuild`, `prebuild:lib`, `prepublishOnly`, `prepare`) is wired to npm lifecycle hooks and runs automatically.
---

# Maintenance & Build Pipelines

There are only **three commands** the maintainer ever needs to run by hand. Everything else (`prebuild`, `prebuild:lib`, `prepublishOnly`, `prepare`) is wired to npm lifecycle hooks and runs automatically.

| Goal | Command |
|---|---|
| Local development (web app + docs) | `npm run dev` |
| Push the docs site to production | `npm run deploy` |
| Publish a new library version to npm | `npm publish` (after bumping the version) |

`docs:gen`, `llms:gen`, `build`, `build:lib`, `prebuild`, `prebuild:lib` are **not** run by hand — they fire automatically (or are only useful for debugging individual steps in isolation).

## 1. Local development (web app + docs)

```bash
npm run dev
```

Expands to: `docs:gen` (regenerates the API JSON) → `vite dev`. Opens the local site with HMR.

## 2. Deploy the docs site (Cloudflare)

```bash
npm run deploy
```

Expands to `npm run build && wrangler deploy`. The `build` step itself is:

```
prebuild  → tsx scripts/gen-api-docs.ts
build     → vite build
postbuild → cp _shell.html → index.html, rm _redirects
```

To preview the production build locally before pushing:

```bash
npm run preview     # build + wrangler dev (local CF emulator)
```

## 3. Publish the library to npm

```bash
npm publish
```

npm automatically chains:

```
prepublishOnly → npm run build:lib
  prebuild:lib → tsx scripts/gen-api-docs.ts
                 tsx scripts/gen-llms-txt.ts
  build:lib    → vite build --config vite.lib.config.ts
                 tsc -p tsconfig.lib.json
```

The published tarball ships: `dist/`, `README.md`, `LICENSE`, `llms.txt`, `llms-full.txt`.

`llms.txt` and `llms-full.txt` are generated artifacts (gitignored) — refreshed on every publish by `scripts/gen-llms-txt.ts`, which runs the same Fumadocs `source` loader as the dev/SSR routes via Vite's programmatic API.

---

# Repository Overview

This repo contains three things in one place, kept together for easier sync and maintenance:

1. The library source code (`src/core`) — the `three-start` library itself.
2. The documentation site content (`content/docs`).
3. The marketing/home site and docs app (built on Fumadocs + TanStack Start).

## Layout

- `src/core/` — library source. Built separately via `vite.lib.config.ts` (`npm run build:lib`).
- `content/docs/` — MDX documentation. Static site generation is already wired up.
- `scripts/gen-api-docs.ts` — parses library sources and emits API type JSON.
- `content/docs/api/generated/` — generated API type data (do not edit by hand).

## Tech Stack

- **App framework:** Fumadocs + TanStack Start (project was scaffolded from this template).
- **Docs format:** MDX in `content/docs/`.
- **API docs:** custom source parser → custom JSON format → rendered with Fumadocs UI's type table.

## Key Commands

- `npm run dev` — dev server.
- `npm run build` — full build. Runs `prebuild` first, which regenerates API type JSON.
- `npm run build:lib` — build the library only (uses `vite.lib.config.ts`).
- `npm run docs:gen` / `npm run prebuild` — regenerate `content/docs/api/generated/` via `scripts/gen-api-docs.ts`.
- `npm run types:check` — type-check the project.
- `npm run lint` / `npm run format` — Biome.

## To Understand the Library

Read these first:

- [content/docs/index.mdx](content/docs/index.mdx)
- [content/docs/why.mdx](content/docs/why.mdx)
- [content/docs/advanced/internals.mdx](content/docs/advanced/internals.mdx)
- The source itself in [src/core/](src/core/)

## Notes for Agents

- When library API surface changes, the generated type JSON in `content/docs/api/generated/` will be refreshed automatically on the next build (or run `npm run docs:gen` manually).
- Do not hand-edit files under `content/docs/api/generated/`.
- Library code lives only in `src/core/`; everything else is the docs/site app.

## API Doc Generation Pipeline

> **Keep this section in sync with the code.** If you change anything that affects how the pipeline works — the extractor's filtering/extraction rules, the JSON schema, the renderer's components, the conventions for authoring source JSDoc — **update the corresponding subsection of this document in the same change**. This file is the contract for future agents and contributors; stale instructions here turn into bugs and wasted re-discoveries.

The API reference pages (`content/docs/api/*.mdx`) combine **hand-written intro prose** with **auto-generated tables** rendered by atomic `Api*` components. The pipeline has three stages:

```
src/core/<Class>.ts  ──┐
                       ├─→  scripts/gen-api-docs.ts  ──→  content/docs/api/generated/<Class>.json
                       │      (ts-morph + shiki)
                       │
content/docs/api/<class>.mdx  ──→  imports JSON  ──→  <ApiInit/ApiProperties/ApiMethods/...>
                                                       (src/components/ApiTable.tsx)
```

### Stage 1 — `scripts/gen-api-docs.ts` (extractor)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vladkrutenyuk/three-start](https://github.com/vladkrutenyuk/three-start) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
