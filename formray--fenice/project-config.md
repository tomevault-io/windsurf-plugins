---
trigger: always_on
description: **FENICE** is an AI-native, production-ready backend platform built on the 2026 Golden Stack. It provides a complete foundation for building REST APIs with authentication, user management, and AI agent discovery via the Model Context Protocol (MCP).
---

# CLAUDE.md — FENICE AI Agent Context

## Project Overview

**FENICE** is an AI-native, production-ready backend platform built on the 2026 Golden Stack. It provides a complete foundation for building REST APIs with authentication, user management, and AI agent discovery via the Model Context Protocol (MCP).

- **Repository:** `https://github.com/formray/fenice`
- **Organization:** Formray
- **Version:** 0.4.0 (latest on `main`)
- **License:** MIT
- **Author:** Giuseppe Albrizio

## Tech Stack

| Layer          | Technology                      | Version |
| -------------- | ------------------------------- | ------- |
| Runtime        | Node.js                         | 22 LTS  |
| Framework      | Hono + `@hono/zod-openapi`      | 4.x     |
| Validation     | Zod v4                          | 4.x     |
| Database       | MongoDB via Mongoose v9         | 9.x     |
| Auth           | JWT (jsonwebtoken + bcryptjs)   | -       |
| Logging        | Pino                            | 10.x    |
| Observability  | OpenTelemetry (auto-instrument) | -       |
| Testing        | Vitest + fast-check             | 4.x     |
| API Docs       | Scalar + LLM markdown           | -       |
| AI Discovery   | MCP (Model Context Protocol)    | -       |
| Language       | TypeScript (strict mode)        | 5.x     |
| Module System  | ESM (`"type": "module"`)        | -       |

## Key Commands

```bash
# Development
npm run dev            # Start dev server (tsx watch + OTel instrumentation)
npm run dev:typecheck  # Typecheck in watch mode

# Quality
npm run lint           # ESLint (src + tests)
npm run lint:fix       # ESLint with auto-fix
npm run format         # Prettier format all
npm run format:check   # Prettier check only
npm run typecheck      # tsc --noEmit
npm run validate       # lint + typecheck + test (run before every PR)

# Testing
npm run test           # Vitest run (single pass)
npm run test:watch     # Vitest interactive watch
npm run test:coverage  # Vitest with v8 coverage

# Build & Production
npm run build          # TypeScript compilation to dist/
npm run start          # Run compiled output (node dist/server.js)

# Shell scripts (four-script pattern)
./setup.sh             # Install deps, create .env, check prerequisites
./dev.sh               # Start MongoDB (Docker) + dev server
./stop.sh              # Stop all Docker services
./reset.sh             # Full clean: node_modules, dist, Docker volumes
```

## Code Style & Conventions

### TypeScript
- Strict mode enabled (`strict: true` in tsconfig.json)
- `exactOptionalPropertyTypes: true` — use `undefined` explicitly for optional props
- `noUncheckedIndexedAccess: true` — indexed access returns `T | undefined`
- `noImplicitReturns: true`
- Target: ES2022, Module: NodeNext

### ESM
- All local imports **must** end in `.js` extension: `import { foo } from './bar.js';`
- `"type": "module"` in package.json
- No CommonJS `require()` — use `import` exclusively

### Naming
- Files: `kebab-case` (e.g., `auth.routes.ts`, `user.model.ts`, `console.adapter.ts`)
- Classes: `PascalCase` (e.g., `AuthService`, `ConsoleEmailAdapter`)
- Variables/functions: `camelCase`
- Constants: `SCREAMING_SNAKE_CASE` for env-derived values
- Schemas: `PascalCase` with `Schema` suffix (e.g., `UserSchema`, `LoginSchema`)

### Commits
- **Conventional Commits** required: `feat:`, `fix:`, `chore:`, `docs:`, `test:`, `refactor:`
- Co-Author line: `Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>`
- Husky + lint-staged pre-commit hooks run ESLint + Prettier automatically

## Architecture

### Request Flow
```
Client Request
  -> Hono Middleware (requestId, requestLogger)
  -> Security (secureHeaders, CORS, bodyLimit, timeout)
  -> Rate Limiter (per-route windows)
  -> API Version (extracts from URL path)
  -> Auth Middleware (JWT verification, on protected routes)
  -> RBAC (role check, on admin-only routes)
  -> OpenAPI Route Handler (Zod validation via @hono/zod-openapi)
  -> Service Layer (business logic)
  -> Mongoose Model (MongoDB operations)
  -> JSON Response (with toJSON transform)
```

### Directory Structure
```
src/
  index.ts              # Hono app setup, route mounting, OpenAPI/Scalar/LLM docs
  server.ts             # Entry point: MongoDB connect, seed admin, @hono/node-server
  instrumentation.ts    # OpenTelemetry NodeSDK (imported via --import flag)
  config/env.ts         # Zod-validated environment variables (76 vars)
  schemas/
    common.schema.ts    # ErrorResponse, Pagination, SuccessResponse
    user.schema.ts      # User, UserCreate, UserUpdate, RoleEnum
    auth.schema.ts      # Login, Signup, AuthTokens, AuthResponse, PasswordReset, EmailVerify
    builder.schema.ts   # BuilderJob, BuilderPlan, BuilderStatus (11 states)
    upload.schema.ts    # UploadSession, UploadChunk
    world.schema.ts     # WorldService, WorldEndpoint, WorldEdge, WorldModel
    world-delta.schema.ts # WorldDeltaEvent discriminated union (9 event types)
    world-ws.schema.ts  # World WS protocol (subscribe, snapshot, delta, ping/pong, error)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [formray/fenice](https://github.com/formray/fenice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
