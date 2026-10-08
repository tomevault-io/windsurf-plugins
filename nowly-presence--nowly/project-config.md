---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Nowly** is an open-source Discord Rich Presence manager (alternative to PreMiD). It detects activity on supported websites via a browser extension and updates Discord Rich Presence through a local native host — no activity data is sent to external servers.

Apps:
- **apps/api** (@nowly/api) - Fastify server: auth, presence registry, assets/CDN, insights ingestion, campaigns, device pairing
- **apps/web** (@nowly/web) - Next.js user dashboard/website (nowly.me)
- **apps/docs** (@nowly/docs) - Next.js public documentation site
- **apps/insights** (@nowly/insights) - Next.js internal analytics dashboard (campaigns, usage views, AI chat over analytics data)
- **apps/extension** (@nowly/extension) - Chromium/Firefox extension that runs presence scripts in the background and talks to the native host
- **apps/discord** (@nowly/discord) - Community Discord bot (status, presence lookup, support, donator roles)
- **apps/native** (Go) - Native messaging host bridging the extension to the Discord desktop client via IPC

Packages:
- **packages/env** - Typed env vars per context (server, client, extension, cli)
- **packages/shared** - Cross-app constants/formatting/schemas
- **packages/analytics** - Shared analytics client/ingestion/consent logic used by extension, web, api, insights
- **packages/ui** - Shared React component library (shadcn-style primitives)
- **packages/locales** - Shared i18n message catalogs
- **packages/sdk**, **packages/cli**, **packages/presences**, **packages/internal-cli** - **git submodules**, each backed by its own GitHub repo (see below)

The flow: a presence is authored against `@nowly/sdk` in `packages/presences` → built/validated by the `nowly` CLI (`packages/cli`) → published to the API/CDN via `packages/internal-cli` → the extension downloads and executes it → the native host relays the activity to Discord.

## Git Submodules

`packages/sdk`, `packages/presences`, `packages/cli`, and `packages/internal-cli` are git submodules, each mirroring a standalone repo under `nowly-presence/*` on GitHub (see root `README.md` "Repositories" table). They are **not** regular workspace folders — commits inside them belong to their own repo/branch (`stable`), not this one.

```bash
git submodule update --init --recursive          # fetch submodule contents after cloning
git submodule update packages/sdk packages/cli packages/presences   # what CI does before lint (does not include internal-cli)
```

When editing files under these paths, check `git status` inside the submodule directory — changes there are a separate commit in a separate repo.

## Monorepo Structure (pnpm workspaces)

```
apps/
  api/           # Fastify server (Node.js)
  web/           # Next.js user dashboard
  docs/          # Next.js public docs
  insights/      # Next.js internal analytics dashboard
  extension/     # Chromium/Firefox extension (Vite-style build via tsx scripts)
  discord/       # Discord bot (Node.js)
  native/        # Native host (Go, cross-platform)

packages/
  env/           # Typed environment variables (t3-oss/env-core)
  shared/        # Shared constants, formatting, zod schemas
  analytics/     # Shared analytics client/ingestion
  ui/            # Shared React component library
  locales/       # Shared i18n catalogs
  sdk/           # [submodule] Presence SDK types & runtime helpers
  cli/           # [submodule] `nowly` CLI - init/build/pack/validate presences
  presences/     # [submodule] Presence source definitions
  internal-cli/  # [submodule] Admin CLI - publishing, R2/CDN sync
```

`pnpm-workspace.yaml` includes `apps/*` and `packages/*`. Use `pnpm --filter @nowly/PACKAGE` to run commands on a specific package. Package manager is pinned via `packageManager` in root `package.json` — check that field rather than assuming a version.

## Development Commands

### Setup
```bash
git submodule update --init --recursive   # required before install if submodules are empty
pnpm install                              # respects pnpm-lock.yaml
```

### Dev servers (root scripts)
```bash
pnpm dev:api                    # Fastify API with tsx watch
pnpm dev:web                    # Next.js web app
pnpm dev:docs                   # Next.js docs site
pnpm dev:insights               # Next.js insights dashboard
pnpm --filter @nowly/extension dev   # extension dev build (watch, canary)
```

### Building
```bash
pnpm build:extension             # production extension build (chrome + firefox)
pnpm build:extension:dev         # dev/canary extension build (chrome)
pnpm --filter @nowly/extension build:chrome    # dev/canary build, chrome
pnpm --filter @nowly/extension build:firefox   # dev/canary build, firefox
pnpm build:macos                 # native host macOS .app/.dmg (apps/native/scripts/macos-release.sh)
pnpm --filter @nowly/api build   # tsup build
pnpm --filter @nowly/sdk build   # tsup build (submodule)
pnpm --filter @nowly/cli build   # tsup build (submodule)
```

### Testing
```bash
pnpm --filter @nowly/api test          # vitest run
pnpm --filter @nowly/api test:watch
pnpm --filter @nowly/env test

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nowly-presence/nowly](https://github.com/nowly-presence/nowly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
