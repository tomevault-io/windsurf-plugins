---
trigger: always_on
description: Self-deployable photography portfolio website built with Next.js 16 + Hono.js + PostgreSQL.
---

# PicImpact

Self-deployable photography portfolio website built with Next.js 16 + Hono.js + PostgreSQL.

## Quick Reference

```bash
# Development
pnpm run dev:server          # Start dev server
pnpm run dev:turbopack       # Dev with Turbopack

# Build
pnpm run build               # Production build
pnpm run build:vercel        # Vercel deployment build
pnpm run build:node          # Node.js deployment build

# Database
pnpm run prisma:generate     # Generate Prisma client
pnpm run prisma:dev          # Run dev migrations
pnpm run prisma:deploy       # Deploy migrations
pnpm run prisma:seed         # Seed initial data

# Linting
pnpm run lint                # ESLint check
pnpm run lint:fix            # Auto-fix lint issues
```

## Tech Stack

- **Framework:** Next.js 16 + React 19 + TypeScript (strict mode)
- **Backend API:** Hono.js (routes in `hono/`)
- **Database:** PostgreSQL + Prisma ORM (`prisma/schema.prisma`)
- **Auth:** better-auth (email/password, TOTP 2FA, WebAuthn Passkeys)
- **Styling:** Tailwind CSS v4 + Radix UI + shadcn/ui
- **State:** Zustand + SWR
- **i18n:** next-intl (zh, en, ja, zh-TW)
- **Storage:** AWS S3 / Cloudflare R2 / Open List API

## Project Structure

```
app/                    # Next.js app router
  (default)/            # Public gallery routes
  admin/                # Admin dashboard
  api/                  # API routes (auth + Hono)
  login/ sign-up/       # Auth pages
components/             # React components (admin/, gallery/, ui/, etc.)
hono/                   # Hono API route handlers
  open/                 # Public API endpoints
  storage/              # Storage-specific endpoints
server/                 # Server-side code
  auth/                 # better-auth config
  db/query/             # Database read queries
  db/operate/           # Database write operations
  lib/                  # Utilities (db client, S3, R2, uploads)
stores/                 # Zustand stores
hooks/                  # React hooks
types/                  # TypeScript type definitions
messages/               # i18n translation JSON files
prisma/                 # Schema, migrations, seed
style/                  # Global CSS
```

## Code Conventions

- **Quotes:** Single quotes (enforced by ESLint)
- **Semicolons:** Never (enforced by ESLint)
- **`any`:** Warned, avoid where possible
- **Path alias:** `~/` maps to project root
- **Components:** PascalCase filenames
- **Pages/routes:** lowercase (Next.js convention)
- **No unused variables/parameters** (TypeScript strict)

## Architecture Notes

- **Database queries:** Read operations in `server/db/query/`, write operations in `server/db/operate/`. Prisma singleton in `server/lib/db.ts`.
- **Server actions:** Shared Next.js server actions in `server/actions/`. Used by page components to deduplicate data fetching.
- **Auth:** better-auth with sessions stored in DB, cookie-based with 7-day duration. Protected routes require auth middleware.
- **Images:** Support EXIF extraction, thumbhash generation, Live Photos, WebGL viewer, map clustering via Supercluster.
- **Deployment:** Docker (multi-arch), Vercel, or bare Node.js. `script.sh` handles Docker startup (migrate + seed + start).

## API Design Specification

### Routing Structure

```
/api/auth/[...all]     → better-auth (login, signup, 2FA, passkeys)
/api/v1/*              → Protected endpoints (require auth session)
/api/public/*          → Public endpoints (no auth required)
```

All `/api/v1/*` and `/api/public/*` routes are handled by Hono via `app/api/[[...route]]/route.ts`.

### Response Format (Target Standard)

All API responses MUST follow this envelope format:

```typescript
// Success response
{
  code: 200,
  message: 'Success',
  data?: T              // Present for GET requests
}

// Error response
{
  code: number,         // HTTP status code (400, 404, 500, etc.)
  message: string       // Human-readable error description
}
```

**Rules:**
- GET endpoints: return `{ code: 200, message: 'Success', data: <result> }`
- Mutation endpoints (POST/PUT/DELETE): return `{ code: 200, message: 'Success' }`
- Errors: throw via helpers in `hono/_lib/errors.ts` (`badRequest`, `notFound`, `conflict`, `serverError`); the global `onError` in `hono/index.ts` shapes the response into the envelope.
- Binary endpoints (image blob, file download): return raw binary with appropriate Content-Type header.

### API Helpers (`hono/_lib/`)

Use these in every Hono handler instead of writing `c.json(...)` by hand:

```ts
import { ok, okEmpty } from '~/hono/_lib/response'
import { badRequest, notFound, conflict, serverError } from '~/hono/_lib/errors'

app.get('/things', async (c) => {
  const data = await fetchThings()
  return ok(c, data)                       // { code: 200, message: 'Success', data }
})

app.put('/things', async (c) => {
  await updateThing(await c.req.json())
  return okEmpty(c)                        // { code: 200, message: 'Success' }
})

app.post('/things', async (c) => {
  const body = await c.req.json()
  if (!body.name) throw badRequest('name is required')
  // ...
})
```

The `sessionMiddleware` in `hono/_lib/context.ts` is mounted on the `/api/v1` chain;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [besscroft/PicImpact](https://github.com/besscroft/PicImpact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
