---
trigger: always_on
description: This file provides guidance to coding agents working in this repository, including Claude Code and Codex.
---

# AGENTS.md

This file provides guidance to coding agents working in this repository, including Claude Code and Codex.

## Build & Development Commands

```bash
pnpm install            # Install dependencies (also installs husky pre-commit hook)
pnpm dev:web            # Start the web UI in Vite dev mode
pnpm build              # Build SPA, then bundle the CLI (writes packages/cli/{dist,web-dist})
pnpm test               # Run tests (Vitest, from root)
pnpm lint               # Biome check (lint + format) — fails on warnings
pnpm lint:fix           # Biome check with auto-fix
pnpm format             # Format code with Biome
pnpm typecheck          # tsc --noEmit across every package (`pnpm -r typecheck`)
```

The package manager is pinned via `packageManager` in `package.json`. Use `corepack enable` if pnpm isn't on your PATH.

### Database (Drizzle ORM + SQLite)

```bash
pnpm db:generate        # Generate a new migration into packages/cli/drizzle/ from schema changes
```

The CLI uses an embedded SQLite database via `better-sqlite3`. There is no separate dev database to start — `getDb()` opens (or creates) the local SQLite file and runs pending migrations on first use.

### Adding UI Components

```bash
cd packages/web && npx shadcn@latest add <component>
```

Components land under `packages/web/src/components/ui/` per `packages/web/components.json`.

## Architecture

**pnpm workspace.** Three packages with real boundaries — no path-alias indirection. The published unit is `packages/cli` (npm name `stagereview`, binary `stagereview`); the rest are private workspace deps that get inlined at build time.

```
pnpm-workspace.yaml         # packages: ["packages/*"]
packages/
  cli/                      # stagereview — published npm package
    src/                    # CLI + local HTTP server (Node, ESM)
      index.ts              # CLI entry (Commander)
      show.ts               # `stagereview show <path>` implementation
      server.ts             # Plain Node http server with regex-compiled routes
      routes/               # API route handlers (one file per resource)
      runs/                 # Chapter run import + processing
      db/                   # Drizzle client, path resolution, schema/
      schema.ts             # Zod schemas for chapter JSON ingestion (strict)
      __tests__/            # Vitest tests
    drizzle/                # Generated SQL migrations + meta journal
    drizzle.config.ts       # Drizzle Kit config
    tsdown.config.ts        # CLI bundler config (inlines @stagereview/types)
  types/                    # @stagereview/types (private, TS-native)
    src/chapters.ts         # Wire-format chapter/key-change schemas + shared HunkReference/LineRef
    src/view-state.ts       # Wire-format view-state schema
    src/index.ts            # Barrel re-export
  web/                      # @stagereview/web (private) — built into ../cli/web-dist
    src/components/         # UI components (shadcn/ui under components/ui/)
    src/lib/                # Frontend utilities + tests
    src/routes/             # SPA route components
    src/styles/             # Tailwind globals
    vite.config.ts          # outDir → ../cli/web-dist
    components.json         # shadcn config
```

### CLI (`packages/cli/src/index.ts`)

Uses [Commander](https://github.com/tj/commander.js) for subcommand parsing. Add new subcommands by chaining `.command(...)` and delegating to a module under `packages/cli/src/`.

### Local Server (`packages/cli/src/server.ts`)

Plain Node `http` server bound to `127.0.0.1`. Route patterns use `:name` placeholders and are compiled to regexes at startup. The server resolves `/api/*` against registered routes and otherwise serves static files from `web-dist/` (next to the bundled CLI) with an `index.html` SPA fallback.

- Route handlers live in `packages/cli/src/routes/` — one file per resource (`runs.ts`, `view-state.ts`, `json.ts`).
- Path traversal is blocked by computing `path.relative(webDist, resolved)` and rejecting any result that escapes the root. **Don't bypass that check** when adding static-serving features.
- The server picks the first free port starting at `5391`. Don't hard-code ports in callers.

### Database Layer (`packages/cli/src/db/`)

- **Client:** `getDb()` in `db/client.ts` is a singleton wrapped around `better-sqlite3`. It enables WAL + foreign keys and auto-runs migrations from `packages/cli/drizzle/`.
- **Schemas:** `db/schema/*.ts`, re-exported from `db/schema/index.ts`. Pass `* as schema` into `drizzle()` so relational queries work.
- **Path:** `db/path.ts` decides where the SQLite file lives (per-OS app data dir).
- Prefer Drizzle's Relational Queries API over the SQL-like query builder unless you need aggregations, custom column selections, or complex joins.

### Shared Types (`packages/types/`)

Wire-format types shared between the CLI's HTTP routes and the SPA. The package exports `.ts` source directly (no compile step) — `tsdown` and `vite` resolve TypeScript natively. The CLI bundle inlines this package via `deps.alwaysBundle` in `tsdown.config.ts`, so the published tarball never has a runtime require for `@stagereview/types`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ReviewStage/stage-cli](https://github.com/ReviewStage/stage-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
