---
trigger: always_on
description: Guidance for LLM-powered agents working in this repository.
---

# AGENTS.md

Guidance for LLM-powered agents working in this repository.

## Project Overview

Moim is a federated events + places service (connpass + foursquare) built with:
- TanStack Start + React for the web app
- Drizzle ORM + PostgreSQL for persistence
- Fedify (`@fedify/fedify`) for ActivityPub federation
- Lingui (`@lingui/core`) for i18n
- Tailwind CSS v4 for styling

## Key Commands

```bash
pnpm dev              # run dev server
pnpm build            # build for production
pnpm start            # run production server
pnpm typecheck        # run tsc --noEmit
pnpm lint             # eslint src/
pnpm db:generate      # generate migration SQL from schema diff
pnpm db:migrate       # apply migrations to app DB
pnpm db:push          # push schema to migration DB
pnpm generate-key     # generate RSA key pair for INSTANCE_ACTOR_KEY
```

## Directory Map

- `src/routes/` — file-based routes (TanStack Start UI pages + co-located API handlers)
  - `src/routes/admin/` — admin panel pages + API handlers
  - `src/routes/auth/` — authentication (OTP, Misskey MiAuth, Mastodon OAuth)
  - `src/routes/events/` — event CRUD, RSVP, dashboards, discussions
  - `src/routes/groups/` — group CRUD, member management, feed
  - `src/routes/places/` — place listing, detail, check-ins, nearby
  - `src/routes/polls/` — poll CRUD, voting
- `src/components/` — reusable React components
  - `src/components/ui/` — primitive UI kit (button, dialog, card, etc.)
  - `src/components/dashboard/` — dashboard layout primitives (StatCard, Pagination, etc.)
  - `src/components/event-form/` — multi-step event creation form
- `src/hooks/` — React hooks (geolocation, event categories, responsive)
- `src/lib/` — client-side utilities (calendar, markdown, place, timezone, utils)
- `src/shared/` — constants shared between client and server (categories, gradients, languages)
- `src/styles/` — global CSS (Tailwind v4)
- `src/scripts/` — CLI scripts (`keygen.ts`)
- `src/server/` — server-side code
  - `src/server/db/` — Drizzle client + schema
  - `src/server/controllers/` — HTTP handlers, organized by domain, wired in `server-entry.ts`
  - `src/server/repositories/` — typed query functions, one file per database table
  - `src/server/services/` — business logic, one file per domain (orchestrates repositories)
  - `src/server/fediverse/` — Fedify federation setup, actor cache, OTP, polls, handles, groups
  - `src/server/storage/` — S3/R2 client
  - `src/server/avatars/` — avatar image processing (sharp)
  - `src/server/events/` — event category helpers (migrating → repositories + services)
  - `src/server/places/` — place find-or-create, categories, audit log, map snapshots (migrating → repositories + services)
  - `src/server/geo/` — H3 hexagonal indexing, reverse geocoding
  - `src/server/i18n/` — Lingui i18n setup + locale catalogs
- `src/server-entry.ts` — h3 app bootstrap: federation middleware, API router, content negotiation

## Layered Architecture

The server codebase follows a layered architecture with strict separation of concerns.
This is an ongoing gradual migration — old patterns coexist with new ones, but all new
code must follow this structure.

### Layers

```
src/server/
  db/
    schema.ts            # Model — Drizzle table definitions
    client.ts            # DB connection
  repositories/          # Repository — typed query functions, one file per entity
  services/              # Service — business logic, orchestrates repositories
  controllers/           # Controller — HTTP handlers, wired in server-entry.ts
    utils.ts             # Shared controller utilities (e.g., optional())
```

### Repository Layer (`src/server/repositories/`)

- **One file per database table** (e.g., `events.ts`, `event-tiers.ts`, `actors.ts`)
- Pure data access — no business logic, no HTTP concepts, no `Response` objects
- Every function has explicit TypeScript input params and return types
- Use Drizzle's `InferSelectModel` / `InferInsertModel` for type definitions
- Import only `db` client and schema — nothing else

```typescript
// src/server/repositories/events.ts
import type { InferSelectModel, InferInsertModel } from "drizzle-orm";
import { db } from "../db/client";
import { events } from "../db/schema";

export type Event = InferSelectModel<typeof events>;
export type NewEvent = InferInsertModel<typeof events>;

export async function findById(id: string): Promise<Event | undefined> {
  const [row] = await db.select().from(events).where(eq(events.id, id)).limit(1);
  return row;
}

export async function insert(values: NewEvent): Promise<Event> {
  const [row] = await db.insert(events).values(values).returning();
  return row;
}
```

### Service Layer (`src/server/services/`)

- **One file per domain** (can span multiple repositories)
- Contains business logic: validation, authorization, orchestration, federation side effects
- Calls repositories for data access — never uses `db` directly
- Defines typed input DTOs and result types for complex operations
- Returns typed results, throws `ServiceError` — never returns `Response`

```typescript
// src/server/services/events.ts
import * as EventRepo from "../repositories/events";
import * as ActorRepo from "../repositories/actors";
import { ServiceError } from "./errors";


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [moim-social/moim](https://github.com/moim-social/moim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
