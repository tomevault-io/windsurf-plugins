---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

MarkLayer — free annotation/collaboration tool for any webpage. Monorepo with a Chrome extension, a Cloudflare Worker backend + web app, a static Astro marketing/SEO site, and an MCP server that bridges annotations to AI coding agents.

## Commands

```bash
bun install          # First-time setup (runs wxt prepare for extension)
bun run dev          # All apps dev mode (Turbo)
bun run build        # Build all
bun run check        # Type-check all (per-workspace `tsc --noEmit`; `astro check` in apps/site)
bun run lint         # Biome lint + format check
bun run lint:fix     # Biome auto-fix

# Releases — run locally; CI only lints, type-checks and builds.
bun run deploy         # Build + deploy worker to Cloudflare (site is embedded in it)
bun run deploy:mcp     # Publish marklayer-mcp to npm, but only if its version is ahead
bun run deploy:site    # Deploy apps/site standalone to Cloudflare Pages
cd apps/site && bun run indexnow:submit   # After a deploy that changed pages: ping IndexNow (Bing, Yandex, …)
bun run zip:extension  # Build Chrome + Firefox store zips into apps/extension/.output

# Extension only
cd apps/extension && bun dev               # Chrome (MV3) dev
cd apps/extension && bun dev:firefox       # Firefox dev
cd apps/extension && bun build             # Chrome build
cd apps/extension && bun build:firefox     # Firefox build
cd apps/extension && bun zip               # Package Chrome zip for store upload
cd apps/extension && bun run zip:all       # Chrome + Firefox zips (what `zip:extension` runs)

# Worker only
cd apps/worker && bun dev          # Vite + Wrangler together
cd apps/worker && bun run build    # vite build, then `embed:site` copies apps/site/dist into public/client
cd apps/worker && bun run deploy   # Build + `wrangler deploy`

# D1 schema changes — write a new file in apps/worker/migrations/, then:
cd apps/worker && bunx wrangler d1 migrations apply marklayer --local   # dev DB
cd apps/worker && bunx wrangler d1 migrations apply marklayer --remote  # prod — `migrations` subcommands
                                                                         # resolve via wrangler.jsonc's
                                                                         # database_name, unlike `d1 execute`
                                                                         # (see below), so "marklayer" is right here

# Marketing site only
cd apps/site && bun dev            # astro dev
cd apps/site && bun run build      # astro build → dist/
cd apps/site && bun run check      # astro check (this workspace's `check`, not tsc)

# Fetch relay only (see apps/fetcher/README.md for hosting)
cd apps/fetcher && bun run dev      # Local relay on :8080 (needs FETCHER_TOKEN)
cd apps/fetcher && bun run build    # → dist/server.js, dependency-free single file
docker build -f apps/fetcher/Dockerfile -t marklayer-fetcher .   # from the repo root

# MCP server only (published to npm as `marklayer-mcp`)
cd apps/mcp && bun run build       # Compile TS → dist/cli.js
cd apps/mcp && bun run dev         # tsc --watch
cd apps/mcp && node scripts/publish-if-new.mjs --check   # Publish preflight, no side effects
```

`bun run test` runs `bun test` in the workspaces that have specs (`packages/types`, `apps/extension`, `apps/worker`, `apps/mcp`, `apps/fetcher`, `packages/emails`, `apps/site`); the rest of the monorepo has none, so `bun run check` (TypeScript) and `bun run lint` (Biome) still carry most of the verification loop. Prefer a test where the failure would be silent — a truncated stream, a guard that stops guarding — over one that restates the type signature.

**`apps/site` is pinned to `typescript@^6`, deliberately.** Its `check` is `astro check`, which needs the TS programmatic API that the 7.x native compiler does not ship yet (withastro/roadmap#1321). The rest of the monorepo runs `tsc --noEmit` on 7.x and is unaffected. Do not bump the site to 7 until `astro check` supports it: the alternative, swapping it for `tsc --noEmit`, silently stops type-checking all 37 `.astro` templates.

## Structure

```
apps/extension/     # Chrome/Firefox extension (WXT + Preact)
  components/       # Canvas, Toolbar, all annotation Layer components
  lib/              # state.ts (signals), renderer.ts, selector.ts, anchor.ts,
                    # portal.ts (Base UI portal target inside the shadow root)
apps/worker/        # CF Worker API + web app (Hono + Vite + Preact)
  src/              # index.ts (Hono routes, robots/llms.txt, /.well-known/*),
                    # api.ts (OpenAPI share API under /api), annotation-room.ts
                    # (Durable Object), proxy.ts (iframe proxy + SSRF guard),
                    # og.ts + og-card.ts + og-marks.ts (share cards), posthog.ts
  web/              # Web app UI (Landing, Viewer, Web* layer components,
                    # useRealtimeSync, useVoiceRoom, signals)
  migrations/       # D1 schema, as ordered `wrangler d1 migrations` files
  wrangler.jsonc    # Worker bindings (D1, DO, R2) + `assets.run_worker_first`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thevrus/MarkLayer](https://github.com/thevrus/MarkLayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
