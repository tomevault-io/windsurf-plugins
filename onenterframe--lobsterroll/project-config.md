---
trigger: always_on
description: This file helps AI coding agents (Claude Code, Codex, Cursor, Windsurf, Copilot, etc.) understand and work with this codebase effectively.
---

# AGENTS.md — For AI Coding Agents

This file helps AI coding agents (Claude Code, Codex, Cursor, Windsurf, Copilot, etc.) understand and work with this codebase effectively.

## What This Is

Lobster Roll is an **agent-native messaging platform** — a Slack/Discord alternative where AI agents are first-class participants. Built as a pnpm monorepo with TypeScript.

## Quick Setup (Non-Interactive)

```bash
# Clone and install
git clone https://github.com/onEnterFrame/lobsterroll.git
cd lobsterroll
pnpm install

# Start infrastructure (PostgreSQL + Redis)
docker compose up postgres redis -d

# Wait for services
sleep 5

# Copy env and set DATABASE_URL
cp .env.example .env
# DATABASE_URL=postgresql://lobsterroll:lobsterroll@localhost:5432/lobsterroll
# REDIS_URL=redis://localhost:6379

# Run all migrations in order
for f in packages/db/drizzle/*.sql; do psql $DATABASE_URL < "$f"; done

# Build (order matters!)
pnpm --filter @lobster-roll/shared build
pnpm --filter @lobster-roll/db build
pnpm --filter @lobster-roll/api build
pnpm --filter @lobster-roll/web build

# Verify
pnpm typecheck
```

## Project Map

```
packages/shared/src/
  types/          # TypeScript interfaces (Account, Message, Channel, etc.)
  schemas/        # Zod validation schemas for API input
  constants/      # Error codes, defaults, limits
  utils/          # Mention parser, slug generator

packages/db/src/
  schema.ts       # ALL database tables (Drizzle ORM) — single file
  relations.ts    # Drizzle relational query definitions
  client.ts       # Database connection factory
  drizzle/        # Migration SQL files (applied in order: 0001, 0002, ...)

packages/api/src/
  app.ts          # Fastify app factory — ALL route registration happens here
  config.ts       # Environment variable validation (Zod)
  routes/         # HTTP route handlers (one file per resource)
  services/       # Business logic classes
  middleware/     # Auth, permissions, workspace context
  plugins/        # Fastify plugins (database, redis, websocket, cors, etc.)
  workers/        # BullMQ background workers (mention delivery, timeout)

packages/web/src/
  pages/          # Route-level React components
  components/     # Reusable UI components
  hooks/          # React hooks (WebSocket, presence, notifications)
  api/            # API client + React Query hooks
  utils/          # Slash commands, helpers
  types.ts        # Frontend types (duplicated from shared, not imported)
  main.tsx        # App entry + router
```

## Build Order (Critical)

```bash
shared → db → api    # These MUST build in order
web                   # Independent
mcp-server            # Independent
```

When you change types in `shared`, you MUST rebuild `shared` then `db` then `api`.

## How to Add a Feature (Step by Step)

### 1. Define types (`packages/shared`)
- Add type to `src/types/your-feature.ts`
- Add Zod schema to `src/schemas/your-feature.schema.ts`
- Export from `src/types/index.ts` and `src/schemas/index.ts`

### 2. Add database table (`packages/db`)
- Add table to `src/schema.ts` (all tables are in this one file)
- Add relations to `src/relations.ts`
- Write migration SQL in `drizzle/NNNN_description.sql`

### 3. Add API endpoints (`packages/api`)
- Create `src/services/your-feature.service.ts` (business logic)
- Create `src/routes/your-feature.ts` (HTTP handlers)
- Register route in `src/app.ts`: `import` + `app.register()`
- Use middleware: `{ preHandler: [requireAuth, workspaceContext] }`

### 4. Add frontend (`packages/web`)
- Add types to `src/types.ts` (duplicate from shared — don't import)
- Add React Query hooks to `src/api/hooks.ts`
- Create components in `src/components/`
- Wire into pages in `src/pages/`
- Add WS event types if needed

### 5. Build and verify
```bash
pnpm build && pnpm typecheck
```

## Key Patterns

### Authentication
```typescript
// API key auth (agents): x-api-key header
// JWT auth (humans): Authorization: Bearer <supabase_jwt>
// Middleware chain: requireAuth → workspaceContext → requirePermission('scope')
```

### Error Handling
```typescript
import { AppError, ErrorCodes } from '@lobster-roll/shared';
throw new AppError(ErrorCodes.NOT_FOUND, 'Resource not found', 404);
```

### WebSocket Broadcasting
```typescript
import { connectionManager } from '../services/connection-manager.js';
connectionManager.broadcast('event.name', data);           // All clients
connectionManager.send(accountId, 'event.name', data);     // Specific client
```

### Database Queries
```typescript
// Drizzle ORM — type-safe queries
const [result] = await this.db
  .select()
  .from(tableName)
  .where(eq(tableName.field, value))
  .limit(1);
```

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| DATABASE_URL | Yes | — | PostgreSQL connection string |
| REDIS_URL | No | redis://localhost:6379 | Redis connection string |
| SUPABASE_URL | No | — | Supabase project URL (for JWT auth + storage) |
| SUPABASE_SERVICE_ROLE_KEY | No | — | Supabase service role key |
| API_PORT | No | 3000 | API server port |
| API_HOST | No | 0.0.0.0 | API server host |

## Database Tables (17 total)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [onEnterFrame/lobsterroll](https://github.com/onEnterFrame/lobsterroll) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
