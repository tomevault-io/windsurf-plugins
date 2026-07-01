---
trigger: always_on
description: **IntelliCircle** is a real-time, location-aware professional networking platform. Users discover hyper-local chat rooms based on GPS proximity and shared interests, join with zero-friction anonymous entry, and optionally upgrade to persistent accounts. The app features AI-powered conversation summaries via Google Gemini and horizontally-scalable WebSocket messaging via Redis Pub/Sub.
---

# CLAUDE.md — IntelliCircle Project Guide

## Project Overview

**IntelliCircle** is a real-time, location-aware professional networking platform. Users discover hyper-local chat rooms based on GPS proximity and shared interests, join with zero-friction anonymous entry, and optionally upgrade to persistent accounts. The app features AI-powered conversation summaries via Google Gemini and horizontally-scalable WebSocket messaging via Redis Pub/Sub.

**Positioning:** "The synchronous, location-aware watercooler" — a real-time complement to LinkedIn's asynchronous model.

---

## Architecture

### Monorepo Structure (npm Workspaces)

```
IntelliCircle/
├── packages/
│   ├── client/          # @intellicircle/client — Next.js 14 App Router frontend
│   ├── server/          # @intellicircle/server — Fastify 5 backend + WebSocket server
│   └── shared/          # @intellicircle/shared — Zod schemas, Drizzle table definitions
├── .github/workflows/   # CI/CD pipelines (CI, Production Deploy, Security Scanning)
├── docker-compose.yml   # Local dev: PostGIS + Redis + Server + Client
├── render.yaml          # Production IaC: Render.com deployment config
├── generate-keys.js     # RSA key pair generator for JWT RS256
└── package.json         # Root workspace config
```

### Service Architecture

| Service | Framework | Port | Purpose |
|---------|-----------|------|---------|
| **Web Server** | Fastify 5 | 8080 | REST API + WebSocket server |
| **Background Worker** | BullMQ | 8080 (dummy) | AI summaries, dead room cleanup |
| **Frontend** | Next.js 14 | 3000 | SSR/CSR React application |
| **Database** | PostgreSQL 15 + PostGIS | 5432 | Primary data store with spatial indexes |
| **Cache/Broker** | Redis (Upstash) | 6379 | Pub/Sub, rate limiting, session blacklist, job queue |

### Data Flow

```
Client (Browser)
  ├── REST API (axios) ──→ Fastify Routes ──→ PostgreSQL (Drizzle ORM)
  └── WebSocket (native) ──→ Fastify WS Handler ──→ Redis Pub/Sub ──→ All connected nodes
                                                  └──→ PostgreSQL (message persistence)
```

---

## Tech Stack Summary

### Frontend (`packages/client`)
- **Framework:** Next.js 14 (App Router), React 18
- **Styling:** Tailwind CSS 3, custom design system (dark theme, Electric Indigo `#4F46E5`)
- **State:** Zustand (auth/UI state, persisted to localStorage), TanStack Query (server state)
- **Animations:** Framer Motion
- **Icons:** lucide-react
- **Chat rendering:** react-virtuoso (virtualized lists)
- **Analytics:** PostHog (product events), Vercel Analytics, Web Vitals
- **Fonts:** Inter (UI), Space Grotesk (display headers)

### Backend (`packages/server`)
- **Framework:** Fastify 5 with TypeScript
- **WebSockets:** @fastify/websocket (native `ws`)
- **Auth:** RS256 asymmetric JWT (@fastify/jwt), Argon2 password hashing
- **Security:** Helmet, CORS, CSRF protection, Redis-backed rate limiting, DOMPurify XSS sanitization
- **Database:** Drizzle ORM → PostgreSQL + PostGIS (`ST_DWithin` spatial queries)
- **Cache/Broker:** ioredis → Redis/Upstash
- **Jobs:** BullMQ (AI summarization, dead room cleanup cron)
- **AI:** Google Gemini 2.5 Flash (@google/generative-ai)
- **Geocoding:** OpenCage API (reverse geocoding with Redis cache, 30-day TTL)
- **Observability:** Pino logger, dd-trace (Datadog APM), hot-shots (StatsD metrics)
- **Monitoring:** Event loop lag (perf_hooks), Redis memory usage, HTTP latency histograms

### Shared (`packages/shared`)
- **Schema:** Drizzle ORM table definitions (pg-core) + Zod validation schemas
- **Exports:** All tables (`users`, `chatRooms`, `messages`, `participants`, `waitlist`, `authAuditLogs`) and all Zod schemas
- **Build:** TypeScript compiled to `dist/`, consumed by both client and server

---

## Key Commands

### Root Level
```bash
npm run dev              # Start both client and server concurrently
npm run dev:client       # Start only the Next.js frontend
npm run dev:server       # Start only the Fastify backend
npm run build:client     # Production build for client
npm run build:server     # Production build for server
npm run check            # TypeScript type check (tsc --noEmit)
```

### Server (`packages/server`)
```bash
npm run dev              # tsx watch src/index.ts (hot reload)
npm run worker           # tsx watch src/worker.ts (BullMQ worker)
npm run build            # tsc compile
npm run start            # node dist/index.js (production)
npm run start:worker     # node dist/worker.js (production worker)
npm run db:push          # Drizzle push schema to DB (rapid prototyping)
npm run db:generate      # Generate SQL migration files
npm run db:migrate       # Run pending migrations
npm run db:enable-postgis # Enable PostGIS extension
npm run seed             # Seed dev data (clears existing!)
npm run seed:production  # Seed production data
```

### Client (`packages/client`)
```bash
npm run dev              # next dev
npm run build            # next build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dharm3112/IntelliCircle](https://github.com/Dharm3112/IntelliCircle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
