---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AphexCMS is a Sanity-inspired, database-agnostic CMS built with SvelteKit V2 (Svelte 5). It uses a monorepo managed by pnpm workspaces and Turborepo.

## Common Commands

```bash
# Development
pnpm dev              # Start studio app (via Turborepo)
pnpm dev:studio       # Start studio app directly
pnpm dev:package      # Start cms-core package only
pnpm build            # Build all packages (Turborepo)
pnpm clean            # Remove build artifacts

# Code Quality
pnpm format           # Format with Prettier
pnpm lint             # Prettier check + ESLint
pnpm check            # Type-check all packages (Turborepo)
pnpm test:package     # Build + type-check cms-core

# Database (proxied to apps/studio)
pnpm db:start         # Start PostgreSQL via Docker
pnpm db:push          # Push schema changes (dev)
pnpm db:generate      # Generate migration files
pnpm db:migrate       # Run migrations (prod)
pnpm db:studio        # Open Drizzle Studio at localhost:4983

# Tests (in apps/studio)
pnpm -F @aphexcms/studio test          # Run all tests
pnpm -F @aphexcms/studio test:watch    # Watch mode
pnpm -F @aphexcms/studio test:ui       # Vitest UI
pnpm -F @aphexcms/studio test:local    # Local API tests only
pnpm -F @aphexcms/studio test:http     # HTTP API tests only
pnpm -F @aphexcms/studio test:graphql  # GraphQL API tests only

# UI Components (adds shadcn-svelte components to @aphexcms/ui)
pnpm shadcn <component-name>

# Template sync (apps/studio → templates/base)
./scripts/sync-template.sh           # Dry run — preview changes
./scripts/sync-template.sh --apply   # Apply changes
```

## Syncing studio → template → CLI

`apps/studio` is the reference app where new features land first. `templates/base` is the starter shipped to end users via `create-aphex`. To flow studio changes downstream:

1. **Sync studio → template** — `./scripts/sync-template.sh --apply`. Template-driven: walks every file tracked in `templates/base/` and copies the matching file from `apps/studio/` if it exists. Skips `src/lib/schemaTypes/**` (template keeps its own minimal example schema). Merges `package.json` preserving the template's `name` and `version`. Studio-only files (tests, seed routes, scripts, render route) never flow over because the template has no matching path.

2. **If studio added a genuinely new file/dir** (e.g. `src/lib/server/cache/`), create a placeholder in `templates/base/` first so the sync picks it up on the next run. This is the tradeoff of the template-driven approach — safe, but requires one manual step for new top-level additions.

3. **Update `templates/base/CHANGELOG.md`** — under `## Unreleased`, list the files that changed and a one-line reason. The template is meant to be customized, so syncs don't auto-apply to downstream projects — the changelog is how users know what to port into their own customized copy. When cutting a release, rename `Unreleased` to the new version.

4. **Push to standalone repo** — when changes to `templates/base/**` land on `main`, `.github/workflows/sync-template.yml` mirrors the folder to `IcelandicIcecream/aphex-base`, rewriting `workspace:*` deps to real versions before pushing. `create-aphex` fetches that repo at scaffold time via `giget` (no bundled templates), so once the workflow runs the new template is live for end users — no `create-aphex` republish needed.

5. **Test the scaffolder locally** — `pnpm -F create-aphex build && node packages/create-aphex/dist/index.js`. By default it pulls `github:IcelandicIcecream/aphex-base`; override with `APHEX_TEMPLATE=github:owner/repo#ref` to test an unpushed branch or a fork.

6. **Verify the template builds** — `pnpm -F @aphexcms/base build`. The template guards module-eval-time env reads with the SvelteKit `building` flag, so build does not require a real `.env`.

The `aphx` CLI (`packages/cli/`, `@aphexcms/cli`) is separate and minimal. Edit `src/index.ts`, run `pnpm -F @aphexcms/cli build`, then `node packages/cli/dist/index.js <cmd>` or `pnpm link --global` to test. The `aphex generate:types` command the template uses is a different bin, exposed by `@aphexcms/cms-core` at `packages/cms-core/src/cli/index.ts`.

## Architecture

### Monorepo Layout

- **apps/studio** — Reference SvelteKit app (the actual CMS admin). Contains schema definitions, auth setup, DB/storage singletons, and API route re-exports.
- **packages/cms-core** — Database-agnostic core engine. Contains admin UI components, route handlers, field validation, schema utilities, plugin system, and TypeScript types. Exports via `@aphexcms/cms-core`, `@aphexcms/cms-core/server`, `@aphexcms/cms-core/client`, `@aphexcms/cms-core/schema`.
- **packages/postgresql-adapter** — PostgreSQL implementation using Drizzle ORM.
- **packages/storage-s3** — S3-compatible storage adapter (R2, AWS S3, MinIO).
- **packages/ui** — Shared shadcn-svelte component library (`@aphexcms/ui`). Components imported as `@aphexcms/ui/shadcn/<component>`.
- **packages/cli, packages/create-aphex** — CLI tooling for scaffolding.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IcelandicIcecream/aphex](https://github.com/IcelandicIcecream/aphex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
