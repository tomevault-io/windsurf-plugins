---
trigger: always_on
description: Working context for tela. Read before contributing. Full architecture is in [`docs/`](docs/) and the deepest subsystem detail in [`docs/architecture.md`](docs/architecture.md); this file is the conventions + hard rules + the things that bite.
---

# CLAUDE.md — tela

Working context for tela. Read before contributing. Full architecture is in [`docs/`](docs/) and the deepest subsystem detail in [`docs/architecture.md`](docs/architecture.md); this file is the conventions + hard rules + the things that bite.

## What tela is

A self-hostable, markdown-native team wiki: Go + PostgreSQL backend, React/TS frontend with a Milkdown editor, live Yjs collaboration, and a TypeScript MCP server so agents are first-class. Public face **https://telawiki.com** (served behind a TLS-terminating reverse proxy; the standalone stack publishes Caddy on `:8780`). `pages.body` is canonical markdown forever — there is **no block table**.

> History: tela was built by an autonomous agent ("forge"), now under conventional development. The old forge workspace is archived out-of-tree (reference only — this repo + git history are the source of truth). Research/design docs from that era live in that archive and were never committed here.

## Layout

- `backend/` — Go. Module `github.com/zcag/tela/backend`, entry `cmd/tela`. `internal/{api,auth,db,mdimport,models,testdb}`. **PostgreSQL** via the `pgx/v5` stdlib driver — DB access is hand-written `database/sql` (positional `$1` placeholders), **no sqlc, no ORM**. Migrations are embedded `NNNN_name.sql` files (forward-only, no down) in `internal/db/migrations`, run automatically by `db.Migrate()` on boot. `0001_init.sql` is a Postgres baseline squashed from the retired SQLite migration history (the live DB held no data worth keeping). Datetimes are TEXT in `'YYYY-MM-DD HH:MM:SS'` UTC (default `tela_now()`), booleans are INTEGER 0/1 — both kept from the SQLite era so Go scans + the frontend `parseSqliteTs` path are unchanged.
- `frontend/` — React 19 + Vite + TS + Tailwind v4 + Radix + Milkdown (`@milkdown/kit`) + TanStack Query + TanStack Router + Orama + cmdk + Lucide + Storybook. `src/{components,lib,routes,styles}` + `App.tsx`/`main.tsx`. State is TanStack Query.
- `mcp/` — **thin stdio↔HTTP proxy** to the backend's built-in MCP server (`/api/mcp`). As of v0.7 the tool/resource surface lives in the Go backend (`internal/api/mcp*.go`), NOT here — this package is a dumb pipe that forwards the MCP protocol over stdio with the PAT as a bearer header (so there's no second implementation to drift). Published as `tela-mcp` on npm. Modern hosts skip it and use HTTP transport directly. See `mcp/README.md` + `docs/mcp-rewrite.md`.
- `deploy/` — docker-compose + `proxy/Caddyfile`. `.env` is gitignored (narrow line, not `*.env`); `.env.example` is committed.
- `landing/` — standalone **marketing landing page** (Astro + Tailwind v4 + OKLCH tokens, self-hosted Geist). Separate static build from the app; `backend/`+`frontend/` are untouched. Locked contracts at repo root: `CONTENT.md` (copy), `DESIGN.md` (look), `ACCEPTANCE.md` (gates). Targets: `make landing-dev` / `landing-build` / `landing-gate`. Tokens in `landing/src/styles/tokens.css` are its own source of truth — never hardcode color/px (the token-conformance gate enforces it). See `landing/README.md`. Caddy serves `landing/dist/` at the apex `/` (the app keeps `/login`, `/spaces`, `/share/*`, `/api/*`); ship it with `make deploy-landing` (split shape: builds, rsyncs `landing/dist/` to the box, reloads the external edge; the standalone stack instead bakes the landing into the proxy image — recreate via `make up`).

## Conventions

- **No issue/task tracker.** Do NOT open GitHub issues (or any other tickets) for this repo, ever. The `#NNN` references in older commits are artifacts from a retired system (forge) — do not continue or imitate them. Commit format is `type(scope): summary` (e.g. `feat(backend): hybrid chunk search`), no issue number. Concise messages, no co-author trailer.
- **Backend:** hand-written SQL via `database/sql`. New migration = new `NNNN_name.sql` (never edit an applied one; forward-only). One handler file per resource in `internal/api`.
- **Frontend hard rules (load-bearing):**
  1. Design tokens in `src/styles/tokens.css`, semantic names only. **Never** hardcode hex / raw px / ad-hoc radii.
  2. Theming via CSS custom properties on `[data-theme="..."]` — runtime switch, no rebuild.
  3. Radix + shadcn-style **owned** components only (`src/components/ui/`). No MUI/Chakra/Mantine/Ant/daisyUI.
  4. `@layer tokens, base, components, utilities` ordering is locked.
  5. Every new UI element uses owned primitives + tokens; missing primitive → build it (with a Storybook story) first.
  6. **Yjs is scoped:** imports allowed ONLY in `src/lib/collab/*` and the collab branch of `milkdown-editor.tsx`. Everything else explores pure-markdown / pure-SQL first.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zcag/tela](https://github.com/zcag/tela) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
