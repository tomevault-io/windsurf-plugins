---
trigger: always_on
description: Guidance for AI coding agents (Claude Code and others) working in this repository.
---

# AGENTS.md

Guidance for AI coding agents (Claude Code and others) working in this repository.

## What this is

Sora UI — an open-source, fully animated React component distribution (shadcn/ui-style registry) built with TypeScript, Tailwind CSS v4, Base UI, Radix UI, and Motion. It's a Turborepo/Bun monorepo. `apps/www` (the docs + registry site) is where almost all component work happens.

## Sora UI Taxonomy

```text
Sora UI
├── Motion       (Animation building blocks: unstyled motion/effects at /motion)
├── Icons        (Animated Lucide icons at /icons)
├── Catalog      (Ready-to-use animated showcases & full layout pages at /catalog)
└── UI           (Base UI + Radix UI foundation infused with Sora Motion & Tailwind CSS at /ui)
```

## Commands

Package manager is **Bun** (`bun@1.3.5`, pinned via `packageManager`). Run from repo root unless noted.

```bash
bun install                 # install deps
bun dev                     # turbo dev, all apps
bun run dev:www             # docs/registry site only (localhost:3000)
bun run build                # turbo build, all apps
bun run check-types          # turbo check-types (tsc --noEmit per package)
bun run lint                 # turbo lint (ultracite check)
bun run format:write          # ultracite fix (biome-based formatter/linter)
bun run registry:build        # rebuild component registry (apps/www); already runs ultracite on generated files
bun run doctor <name>         # targeted component check (registry, demoProps, MDX, meta.json)
```

Single-app / targeted commands (run inside `apps/www`, or use `--filter=www`):

```bash
cd apps/www
bun run dev                  # next dev -p 3000
bun run check-types           # tsc --noEmit
bun run lint                  # ultracite check
bun run lint:links             # validates internal doc links (also runs in pre-commit for content/**)
bun run registry:build        # merges registry-item.json, builds public/r/*.json, ultracite-fixes generated files
```

**Windows note:** `npx biome` / `npx tsc` resolve to unrelated decoy npm packages in this repo and silently produce fake output. Always invoke the real binaries directly: `node_modules/.bin/biome.exe check <path>` and `node_modules/.bin/tsc.exe --noEmit -p apps/www` (or `apps/www/tsconfig.json`).

There is no test runner configured in this repo — verification is via `check-types`, `lint`, `registry:build`, and `bun run doctor <name>` (for targeted component health checks; avoid `doctor --all` during feature work). `registry:build` already runs `ultracite fix` on the files it generates (`apps/www/__registry__/*`, `public/r/*.json`). Do not run a second `bun x ultracite fix` / `bun run format:write` just because you ran `registry:build`.

Git hooks (lefthook): pre-commit runs `ultracite fix` on staged JS/TS/JSON/CSS and `lint:links` on `apps/www/content/**`; pre-push runs `bun run build`.

## Architecture

### Monorepo layout

```
apps/
  www/   — docs site + component registry (Next.js, Fumadocs). Primary work surface.
  xmcp/  — MCP server (xmcp) exposing Sora docs/registry as tools (search docs, list components, get component info) for AI assistants.
packages/
  ui/                 — @workspace/ui: shared primitives-adjacent utilities (cn, get-strict-context, get-motion-component), globals.css, base hooks.
  auth-ui/            — @workspace/auth-ui: better-auth UI wiring.
  db/                 — @workspace/db: Drizzle ORM schema/client (db:generate/push/migrate/studio).
  typescript-config/  — shared tsconfig bases.
  www-cli/            — @workspace/www-cli: internal contributor CLI for scaffolding registry components (`bun run create`).
```

### The registry system (apps/www/registry) — the core of this repo

Everything under `apps/www/registry` follows the shadcn/ui registry-item.json convention and is described in detail in `apps/www/registry/README.md` and `CONTRIBUTING.md`.

```
registry/
  ui/
    base/{name}/                                                 — Base UI + Motion components (UI tier)
    radix/{name}/                                                — Radix UI + Motion components (UI tier)
  primitives/
    {animate|buttons|disclosure|effects|texts}/{name}/           — Unstyled animation primitives (Motion tier)
  catalog/{name}/                                                — Ready-to-use animated layout showcases (Catalog tier)
  icons/{name}/                                                  — Animated Lucide icons (@soralabs/icons-*)
  demo/
    ui/{base|radix}/{name}/                                      — Manual demos for UI tier
    primitives/{category}/{name}/                                — Manual demos for Motion tier
    catalog/{name}/                                              — Showcase preview demos for Catalog tier
    icons/{name}/                                                — Interactive preview demos for Icons tier
  hooks/, lib/
```

### content/ — distinct content trees, don't conflate them

- `content/docs/` — the core documentation site (routed at `/docs`). Top-level guide pages.
- `content/motion/` — unstyled animation primitives (routed at `/motion`). Flat, one MDX per primitive.
- `content/icons/` — animated icons documentation and showcase (routed at `/icons`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SoraLabsOSS/ui](https://github.com/SoraLabsOSS/ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
