---
trigger: always_on
description: This document contains guidelines and commands for agentic coding agents working in the read-vagabond repository.
---

# AGENTS.md

This document contains guidelines and commands for agentic coding agents working in the read-vagabond repository.

## Project Overview

This is a manga reader application built with Astro, serving Takehiko Inoue's Vagabond manga with a minimalist, high-quality reading experience. The reader is a **fully static site** (Astro SSG): all pages are pre-rendered at build time from a local SQLite database, and manga images are served from Bunny CDN.

A separate Cloudflare Worker (`mihon/`) exposes a small read-only API consumed by the Mihon/Keiyoushi browser extension. This is the only part of the project still running on Cloudflare.

> **Note:** This project was migrated off Cloudflare Workers/Pages (SSR, D1, R2). If you find references to D1, R2, the Cloudflare adapter, `Astro.locals.runtime`, or cache middleware in older docs or code, they are legacy and should not be reintroduced in the main app.

## Build & Development Commands

### Core Commands

```bash
# Development
pnpm dev              # Start Astro dev server
pnpm build            # Build the static site to dist/
pnpm preview          # Preview the production build locally

# Astro CLI
pnpm astro <command>  # Run any Astro CLI command
```

### Database Commands

The reader reads from a local SQLite file (`local.db`, committed to the repo) via Drizzle ORM. Schema is defined in `src/db/schema.ts`; the Drizzle client is in `src/db/client.ts`.

```bash
# Generate a migration from schema changes (src/db/schema.ts)
pnpm drizzle-kit generate

# Apply migrations to local.db
pnpm drizzle-kit migrate

# Inspect/browse the database
pnpm drizzle-kit studio
```

Seeds live in `seeds/` and are applied directly to the SQLite file:

```bash
sqlite3 local.db < seeds/0000_seed_vagabond_metadata.sql
```

Because the static build reads `local.db` at build time, the database file is committed so builds are reproducible. Regenerate/seed it locally if you change the schema or data, then commit the updated `local.db`.

### Testing

No dedicated test framework is currently configured. Manual testing should be performed by:

1. Running `pnpm dev` and testing all user flows
2. Running `pnpm build && pnpm preview` to validate the static output
3. Verifying responsive design across devices and both light/dark themes

## Architecture & Tech Stack

- **Framework**: Astro v6 with `output: "static"` (SSG)
- **Database**: Local SQLite (`local.db`) accessed at build time via Drizzle ORM + `@libsql/client`
- **Migrations**: drizzle-kit, output to `drizzle/migrations/`
- **Image CDN**: Bunny CDN (`https://vagabond.b-cdn.net/`)
- **Styling**: Tailwind CSS v4 (via `@tailwindcss/vite`) with Flowbite components
- **State**: nanostores (reader UI state)
- **Mihon API**: separate Cloudflare Worker (`mihon/`) built with Hono, backed by D1
- **Package manager**: pnpm workspace (`pnpm-workspace.yaml`) — root reader app + `mihon` worker

## Code Style Guidelines

### File Organization

```txt
src/
├── pages/                          # Astro file-based routing (all statically rendered)
│   ├── index.astro                 # Homepage
│   ├── 404.astro
│   ├── sitemap.xml.ts              # Generated sitemap
│   ├── volume-[volume]/            # Volume + chapter routes (getStaticPaths)
│   └── volume-[volume]/chapter-[chapter]/
├── layouts/                        # BaseLayout, MainLayout, ChapterLayout
├── components/                     # UI components (.astro)
├── feature/reader/store.ts         # nanostores reader state
├── lib/                            # Data access (db.ts), page URLs (page.ts), metadata
├── db/                             # Drizzle: schema.ts + client.ts
├── styles/                         # Global styles and Tailwind imports
└── env.d.ts                        # Astro client type reference

drizzle/migrations/                 # Generated SQL migrations
seeds/                              # SQL seed data for local.db
mihon/                              # Cloudflare Worker (Hono) for the extension API
manga/                              # Image source + bunny-upload.sh (rclone -> Bunny CDN)
```

### Imports

- Use relative imports for internal modules: `import { getMangaVolumes } from "../lib/db"`
- Use absolute imports for dependencies: `import type { GetStaticPaths } from "astro"`
- Group imports: 1) Astro imports, 2) third-party, 3) local imports
- Type imports use `import type` when possible

### TypeScript Patterns

- Strict TypeScript config (extends `astro/tsconfigs/strict`)
- Prefer letting Drizzle infer query result types; add explicit annotations for function parameters and returns
- Leverage Astro's built-in types: `GetStaticPaths`, `APIRoute`, `AstroGlobal`

### Astro Components

- Use frontmatter (---) for build-time logic and data fetching
- Pages with dynamic params (`[volume]`, `[chapter]`) **must** export `getStaticPaths` so every page is pre-rendered
- Use HTML5 semantic elements appropriately
- Include proper meta tags for SEO and accessibility (see `MetadataTags`)
- Use `Astro.redirect("/")` for invalid params

### Database Operations (Drizzle ORM)

- Import the shared client from `src/db/client.ts` (`db`) and tables from `src/db/schema.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crydafan/read-vagabond](https://github.com/crydafan/read-vagabond) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
