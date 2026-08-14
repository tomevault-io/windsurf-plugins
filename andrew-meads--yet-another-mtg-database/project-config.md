---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A personal Magic: The Gathering card database and collection manager. Next.js 16 (App Router, React 19) frontend + API routes, backed by MongoDB via Mongoose. Card data originates from Scryfall bulk JSON. Features include Scryfall-style search, collection and deck management, drag-and-drop card organization, and camera-based card scanning (proxied to an external card-scanner backend).

## Planning & implementation checklist

Any non-trivial plan must include steps for:
1. **Tests** — add or update unit, integration, component, and/or e2e tests as appropriate for the change. Do not skip this even if the task description doesn't mention it.
2. **Lint** — run `npm run lint` at the end and fix all errors and warnings before considering the task done.

## Commands

```bash
npm run dev              # Start dev server (http://localhost:3000)
npm run build            # Production build
npm run start            # Run production build
npm run lint             # ESLint (eslint-config-next)

npm run init-db          # Import Scryfall bulk card data into MongoDB
npm run whitelist-user   # Whitelist a user by email (see Auth below)

npm test                 # Run all Vitest projects (unit + integration + components)
npm run test:unit        # Pure-logic unit tests (search engine, sort, helpers)
npm run test:integration # API route + server-helper tests (mongodb-memory-server)
npm run test:components  # React component/hook/context tests (jsdom + RTL)
npm run test:coverage    # Vitest with v8 coverage
npm run test:e2e         # Playwright E2E (run `npm run test:e2e:install` once first)
```

### Testing

The suite (added with Vitest 4 + Playwright) lives in three Vitest projects plus E2E:
- **`unit`** (node) — pure functions: `src/lib/search/**`, `src/lib/sortConfig.ts`, `grouping.ts`, etc. Tests co-located in `__tests__/`.
- **`integration`** (node) — API route handlers and `src/lib/server/**` against an in-memory MongoDB (`mongodb-memory-server`); `getServerSession` is mocked. Files in `tests/integration/`; shared lifecycle/auth/seed helpers in `tests/integration/setup.ts` and `helpers.ts`. Runs single-fork with isolation off so one DB connection is shared.
- **`jsdom`** (jsdom) — components, hooks, contexts via Testing Library + MSW. Global mocks (`next/navigation`, `next/image`, `matchMedia`) in `vitest.setup.jsdom.ts`.
- **E2E** (`e2e/`, Playwright) — runs the app on port 3100 with its own `distDir` (`E2E_DIST_DIR`, so it never collides with a running dev server). `e2e/global-setup.ts` starts a fixed-port in-memory Mongo, seeds a user + active "Main Collection" + cards, and mints a NextAuth session cookie (no production auth changes).

Config: `vitest.config.ts`, `playwright.config.ts`. Run a single project with `vitest run --project <name>`.

### Local development infrastructure

`docker-compose-dev.yml` spins up only the backing services — MongoDB (host port `27017`), the card-scanner Postgres DB, and the external `card-scanner` backend (host port `8000`) — **without** the Next.js app. Use it for developing the app locally with `npm run dev` (which connects to Mongo via `MONGO_DB_URI` in `.env`, default `mongodb://127.0.0.1:27017/...`):

```bash
docker compose -f docker-compose-dev.yml up -d   # start databases + scanner
npm run dev                                       # run the Next.js app on the host
```

`docker-compose.yml` is the full production stack (app + Mongo + scanner behind Caddy); `docker-compose-dev.yml` is the same minus the app service and reverse proxy.

### Seeding the database

`init-db` (`src/scripts/init-db.ts`) streams a large Scryfall "oracle/all cards" JSON file into the `cards` collection. It is a `commander` CLI:

```bash
npm run init-db -- -f bulk-data/oracle-cards-XXXX.json   # import from file
npm run init-db -- --data-url <url>                       # download + import
npm run init-db -- -f <file> --clear                      # wipe cards first
```

Place bulk JSON files in `bulk-data/` (gitignored). Scripts run via `tsx` and load env through `dotenv/config`, reading `MONGO_DB_URI`.

### Whitelisting a user

Sign-in is **deny-by-default**: only emails present in the `users` collection can log in (see Auth). Add one with:

```bash
npm run whitelist-user -- user@example.com
```

## Environment

Copy `.env.example` to `.env`. Key vars: `MONGO_DB_URI`, `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` / `AUTH_SECRET` / `NEXTAUTH_URL` (NextAuth Google OAuth), `SCANNER_BASE_URL` (external card-scanner backend the `/api/scan` route proxies to — defaults to `http://localhost:8000`), `ALL_CARDS_FILE` (default bulk import path), `SCRYFALL_API_BASE_URL` (Scryfall API base, defaults to `https://api.scryfall.com`; used by the card-refresh and set-icon routes).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andrew-meads/yet-another-mtg-database](https://github.com/andrew-meads/yet-another-mtg-database) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
