---
trigger: always_on
description: Race timing system. RFID readers detect participants crossing start/finish gates.
---

# LeszyRun — Development Guide for Claude

## Project summary

Race timing system. RFID readers detect participants crossing start/finish gates.
Events are published via MQTT. Backend processes them and stores results in local
PostgreSQL. Syncs to Supabase when online. See ARCHITECTURE.md for full design.

## Hardware documentation

- `docs/impinj-r700-api/endpoints.md` — known R700 REST API endpoints (system status, antennas, MQTT, inventory presets)
- `docs/impinj-r700-api/mqtt.md` — full `MqttConfiguration` schema with all field names, types, constraints, and common wrong-name mistakes
- `docs/impinj-r700-api/inventory-preset-fields.md` — inventory preset field reference: `transmitPowerCdbm` range, `inventorySession` values, `inventorySearchMode` options, `estimatedTagPopulation` sizing, `rfMode` IDs (note: R700 does NOT support mode 1000)

## Language and runtime

- **JavaScript only** — no TypeScript. Plain JS with JSDoc where helpful.
- Node.js backend (Fastify)
- React frontend (Vite)

## Stack decisions (do not change without discussion)

- ORM: **Drizzle** (not Prisma, not raw SQL)
- Backend framework: **Fastify** (not Express)
- Frontend build: **Vite** (not CRA, not Next.js)
- UI: **shadcn/ui** + **Tailwind v4**
- Tables: **TanStack Table v8** (inline editing)
- Server-side real-time: **WebSockets** via `ws` npm package (not Socket.io)
- Database: **PostgreSQL 16** in Docker with named volume `pgdata`
- MQTT client: `mqtt` npm package
- Supabase sync: `@supabase/supabase-js`
- PDF export: `pdf-lib`

## Monorepo structure

```
LeszyRun/
  backend/      Node.js + Fastify
  frontend/     React + Vite (admin UI)
  public/       React + Vite (public-facing: live results, volunteer bib entry, self-service check-in)
  packages/ui/  Shared UI components (@leszyrun/ui)
  mosquitto/    native macOS, NOT dockerized (hardware constraint)
```

**`packages/ui/` rule:** All race result rendering (status badges, position estimation, podium, results tables) MUST use shared components from `@leszyrun/ui`. Never duplicate result display logic in `frontend/` or `public/` — if a component is missing, add it to `packages/ui/` first, then import in both apps.

## Running locally

```bash
# Start Mosquitto (native, from project root)
/opt/homebrew/sbin/mosquitto -c mosquitto/config/mosquitto.conf

# Start everything else
docker compose up
```

Admin frontend: http://localhost:3000
Backend API: http://localhost:3001
PostgreSQL: localhost:5432

Public app (landing page + kalendarz) — run separately:
```bash
cd public && npx vite --port 3002
```
Public app: http://localhost:3002

## Environment variables

Backend (set in docker-compose.yml or .env):
- `DATABASE_URL` — postgres connection string
- `MQTT_URL` — mqtt://host.docker.internal:1883
- `PORT` — default 3001
- `SUPABASE_URL` — optional, sync disabled if missing
- `SUPABASE_SERVICE_ROLE_KEY` — service_role key (NOT anon/publishable), bypasses RLS. Backend only. Sync disabled if missing.

Frontend:
- `VITE_API_URL` — http://localhost:3001
- `VITE_WS_URL` — ws://localhost:3001

SMS (backend, optional — SMS disabled if missing):
- `SMSAPI_TOKEN` — API token for SMSAPI.pl
- `SMSAPI_SENDER` — sender name registered with SMSAPI

Scraper (backend, optional — URL resolver disabled if missing):
- `BRAVE_SEARCH_API_KEY` — API key from https://brave.com/search/api/ (1000 queries/month free tier)

## Backend conventions

- All routes in `src/routes/`, one file per resource
- Register routes in `src/server.js` with prefix (`/api/events`, `/api/participants`, etc.)
- Drizzle schema in `src/db/schema.js`, client in `src/db/index.js`
- Use Drizzle migrations (`drizzle-kit generate` + `drizzle-kit migrate`)
- **Migrations MUST be registered in `src/db/migrations/meta/_journal.json`** — Drizzle ignores SQL files not listed there. When writing a migration manually: create the `.sql` file AND add an entry to the journal (`idx`, `version: "7"`, `when`, `tag` matching filename without `.sql`, `breakpoints: true`). If a migration already ran (backend started and logged "Migrations complete"), do NOT modify the SQL file — create a new numbered file + new journal entry instead.
- **DDL changes MUST be applied to both local DB and Supabase.** Local DB uses Drizzle migrations (auto-run on backend start). Supabase must be updated separately via `mcp__supabase__apply_migration`. Every schema change (new table, alter column, add index, etc.) requires both — never apply to one without the other.
- WebSocket broadcaster in `src/ws/broadcaster.js` — export a `broadcast(event, data)` function
- MQTT client starts on server boot, crossing detector subscribes to it
- Supabase sync runs as a `setInterval` in background, does not block requests
- All IDs are UUIDs (`gen_random_uuid()` in Postgres, `crypto.randomUUID()` in JS)
- Timestamps as `timestamptz` in DB, ISO strings in API responses

## API conventions

- REST: `GET /api/:resource`, `POST /api/:resource`, `PATCH /api/:resource/:id`, `DELETE /api/:resource/:id`
- Always return `{ data }` wrapper for single items, `{ data: [] }` for lists
- Errors: `{ error: 'message' }` with appropriate HTTP status
- PATCH uses partial updates — only send changed fields

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [derberg/leszy.run](https://github.com/derberg/leszy.run) — distributed by [TomeVault](https://tomevault.io/claim/derberg).
<!-- tomevault:4.0:windsurf_rules:2026-04-18 -->
