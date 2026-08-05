---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Dockstash — Claude Code Guide

## 1. Project Overview

Dockstash is **open-source, self-hosted backup orchestration for Docker**: encrypted restic snapshots on storage the operator owns, a pull-based agent fleet, schedules, restore drills, health monitoring and alerting. It is organized as feature modules on both client and server. The client runs React 18 with Redux Toolkit and Vite; the server runs Express with Mongoose 8. Both sides use TypeScript. A third package, `agent/`, is the VPS daemon.

**This repository contains no billing code**, no per-account limits, and no telemetry — see §9. A managed service (Dockstash Cloud, dockstash.com) runs the same codebase; its metering and subscription layer lives outside this repo and must never be added here. Licensed **AGPL-3.0-only**, with a commercial license available separately — which is why contributions require the CLA in `.github/CLA.md`.

## 2. Stack

**Client**
- React 18, ReactDOM 18
- Redux Toolkit 2, react-redux 9
- React Router 6 (`createBrowserRouter`)
- Native `fetch` via `shared/api/client.ts` (no axios)
- Vite 6, TypeScript 5, Sass
- Socket.IO Client 4
- Vitest + React Testing Library for tests
- zod for form/API validation

**Server**
- Express 4, TypeScript 5
- Mongoose 8
- Passport (local + JWT strategies via `passport-jwt`)
- jsonwebtoken 9, bcryptjs
- Socket.IO 4
- pino / pino-http for structured logging
- zod for env validation (`src/config/env.ts`)
- Vitest + supertest for integration tests; `mongodb-memory-server` for in-process MongoDB

## 3. Repo Layout

```
dockstash/
├── client/
│   └── src/
│       ├── app/              # Store (store.ts), router (router.tsx), providers
│       ├── features/         # auth/, dashboard/, projects/, snapshots/, fleet/, health/, settings/, profile/, guide/, admin/
│       │   └── <name>/
│       │       ├── components/
│       │       ├── store/        # slice.ts, thunks.ts, selectors.ts
│       │       ├── api.ts        # Feature-scoped fetch calls via apiClient
│       │       ├── types.ts
│       │       ├── routes.tsx    # Route array exported for router.tsx
│       │       └── index.ts      # Public API — named re-exports only
│       └── shared/
│           ├── api/client.ts     # Typed fetch wrapper (ApiError, apiClient<T>)
│           ├── hooks/redux.ts    # useAppSelector / useAppDispatch typed hooks
│           └── utils/cookies.ts  # saveCookie / loadCookie / removeCookie
├── server/
│   └── src/
│       ├── app.ts            # createApp() — Express wiring, middleware, route mounts
│       ├── server.ts         # Entry point — connects DB, starts HTTP server
│       ├── config/           # env.ts (zod), db.ts, passport.ts, logger.ts
│       ├── modules/          # auth/, users/, backups/, scheduler/, agents/, alerting/, health/, audit/, recovery/, settings/, legal/, communication/
│       │   └── <name>/
│       │       ├── <name>.controller.ts
│       │       ├── <name>.service.ts
│       │       ├── <name>.model.ts       # Mongoose model + InferSchemaType
│       │       ├── <name>.schema.ts      # zod request-body schemas
│       │       ├── <name>.routes.ts
│       │       ├── <name>.test.ts
│       │       └── index.ts
│       └── shared/
│           ├── middleware/   # error-handler, not-found, require-auth, request-id
│           ├── testing/      # mongo.ts (startMemoryMongo / stopMemoryMongo / clearCollections)
│           ├── types/        # express.d.ts augmentations
│           └── utils/        # http-error.ts (HttpError), async-handler.ts
├── .claude/                  # Claude Code configuration (see section 6)
├── .env                      # Environment variables (single source of truth)
├── .env.example              # Template — copy this to .env
└── .gitignore
```

## 4. How to Run

**ALL commands run inside Docker — never on the host.** See
`.claude/rules/docker-only-commands.md` (strict rule). Dev tooling uses
`node:24-bookworm` for CI parity; the stack runs via `docker compose`.

```bash
# Full stack (api, web, mongo, redis, docker-socket-proxy)
docker compose up -d --build
docker compose ps                    # all services should report `healthy`

# Install dependencies
docker run --rm -v "$PWD/client":/app -w /app node:24-bookworm npm install
docker run --rm -v "$PWD/server":/app -w /app node:24-bookworm npm install

# Type-check both sides
docker run --rm -v "$PWD/client":/app -w /app node:24-bookworm npm run typecheck
docker run --rm -v "$PWD/server":/app -w /app node:24-bookworm npm run typecheck

# Lint
docker run --rm -v "$PWD/client":/app -w /app node:24-bookworm npm run lint
docker run --rm -v "$PWD/server":/app -w /app node:24-bookworm npm run lint

# Run all tests
docker run --rm -v "$PWD/server":/app -w /app node:24-bookworm npm test
docker run --rm -v "$PWD/client":/app -w /app node:24-bookworm npm test

# Run a single test file
docker run --rm -v "$PWD/server":/app -w /app node:24-bookworm \
  npx vitest run src/modules/auth/auth.test.ts
docker run --rm -v "$PWD/client":/app -w /app node:24-bookworm \
  npx vitest run src/features/auth/store/slice.test.ts

# Production build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SelmiAbderrahim/dockstash](https://github.com/SelmiAbderrahim/dockstash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
