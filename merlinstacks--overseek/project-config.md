---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

OverSeek v2 is a self-hosted WooCommerce command center (analytics, unified inbox, AI co-pilot, inventory management). It is a monorepo with a React 19 frontend, Fastify 5 backend, and a WordPress/WooCommerce plugin.

## Commands

All commands assume you are running inside Docker containers (the standard dev environment).

### Development

```bash
# Start both client and server concurrently (from repo root)
npm run dev

# Start server only
cd server && npm run dev

# Start client only
cd client && npm run dev
```

### Building

```bash
cd server && npm run build          # TypeScript compilation
cd server && npm run build:typecheck # Type check only
cd client && npm run build          # Vite production build
```

### Linting

```bash
cd server && npm run lint
cd client && npm run lint
```

### Testing

```bash
# Run all server tests
docker exec overseekv2-api-1 npx vitest run

# Run a specific test file
docker exec overseekv2-api-1 npx vitest run src/services/__tests__/MyService.test.ts

# Watch mode
docker exec -it overseekv2-api-1 npx vitest

# Client tests
cd client && npm run test:run
```

### Database

```bash
# Apply schema changes (development)
docker exec overseekv2-api-1 npx prisma db push

# Regenerate Prisma client after schema changes
docker exec overseekv2-api-1 npx prisma generate

# Open database GUI
docker exec -it overseekv2-api-1 npx prisma studio

# Run migrations (from repo root)
npm run db:migrate
npm run db:generate
```

## Architecture

### Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 19, Vite, TypeScript, Tailwind CSS v4 |
| Backend | Fastify 5, Node.js 22+, TypeScript |
| ORM | Prisma 7 (PostgreSQL 17 + pgvector) |
| Search | Elasticsearch 9.x |
| Cache/Queue | Redis 7 + BullMQ |
| Real-time | Socket.IO 4.8 |
| Testing | Vitest |
| AI | OpenRouter (proxies to various LLMs) |

### Data Flow

```
WooCommerce Store → WordPress Plugin → Fastify API → PostgreSQL / Elasticsearch / Redis → React Frontend
```

The WordPress plugin (`overseek-wc-plugin/`) syncs WooCommerce data to the platform via REST API calls. The Fastify API serves the React frontend and handles all business logic.

### Backend Structure

- `server/src/routes/` — Fastify route plugins (registered in `app.ts` with `/api/` prefix)
- `server/src/services/` — Business logic, one service per domain
- `server/src/workers/` — BullMQ worker registration and SIGTERM shutdown
- `server/src/services/queue/QueueFactory.ts` — Queue creation/worker factory
- `server/src/services/scheduler/` — Cron-like schedulers (BullMQ repeatable + setInterval)
- `server/src/middleware/` — Auth, validation, tracking
- `server/src/utils/` — Shared utilities (prisma, logger, redis, elastic)
- `server/prisma/schema.prisma` — Single schema file for all 80+ models

### Frontend Structure

- `client/src/pages/` — Route-level components (lazy-loaded)
- `client/src/components/[feature]/` — Feature-specific components
- `client/src/components/ui/` — Reusable primitives (Dialog, EmptyState, etc.)
- `client/src/context/` — React contexts (AuthContext, AccountContext, SocketContext)
- `client/src/hooks/` — Custom hooks
- Route registration is in `client/src/App.tsx` — add lazy import + `<Route>` inside `DashboardLayout`

## Critical Patterns

### Multi-tenancy (MOST IMPORTANT)

Every Prisma model storing account data **must** have `accountId`. Every query **must** filter by `accountId`. Every mutation must verify ownership before updating/deleting.

```typescript
// ALWAYS scope queries to accountId
const items = await prisma.item.findMany({ where: { accountId } });

// ALWAYS verify ownership before mutation
const existing = await prisma.item.findFirst({ where: { id, accountId } });
if (!existing) return reply.code(404).send({ error: 'Not found' });
```

### API Routes

Routes are Fastify plugins in `server/src/routes/`. Auth is applied via `preHandler: requireAuthFastify`. This provides `request.user` and `request.accountId` (from `X-Account-ID` header). Validate all input with Zod using `.safeParse()`.

```typescript
const featureRoutes: FastifyPluginAsync = async (fastify) => {
    fastify.get('/', { preHandler: requireAuthFastify }, async (request, reply) => {
        const accountId = request.accountId;
        if (!accountId) return reply.code(400).send({ error: 'Account context required' });
        // ...
    });
};
export default featureRoutes;
```

Register in `server/src/app.ts`: `app.register(featureRoutes, { prefix: '/api/feature' });`

### Frontend Data Fetching

Use the `useApiQuery` and `useApiMutation` hooks (wraps `@tanstack/react-query`) for data fetching in components. These hooks handle auth headers automatically.

```tsx
import { useApiQuery, useApiMutation } from '../hooks/useApiQuery';

// Query
const { data, isLoading, error } = useApiQuery({
    queryKey: ['endpoint'],
    queryFn: async () => {
        const res = await fetch('/api/endpoint', {
            headers: {
                'Authorization': `Bearer ${token}`,
                'X-Account-ID': selectedAccount.id,
            }
        });
        return res.json();
    }
});


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MerlinStacks/overseek](https://github.com/MerlinStacks/overseek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
