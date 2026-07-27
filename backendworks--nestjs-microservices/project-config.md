---
trigger: always_on
description: Four independent NestJS apps plus two versioned shared database packages, all behind a **Kong API Gateway**.
---

# Copilot Instructions – NestJS Microservices

## Architecture Overview

Four independent NestJS apps plus two versioned shared database packages, all behind a **Kong API Gateway**.

```
Client → Kong (:8000) → auth-service (:9001) / post-service (:9002)
                              ↕ gRPC (:50051 / :50052)

auth-service  ──uses──► @backendworks/auth-db  (npm package, GitHub Packages)
auth-worker   ──uses──► @backendworks/auth-db
post-service  ──uses──► @backendworks/post-db  (npm package, GitHub Packages)
post-worker   ──uses──► @backendworks/post-db

Both packages internally use Prisma but expose an ORM-agnostic repository interface
so the underlying ORM can be swapped without touching any service.
```

### Apps and their repositories

| App            | HTTP port | gRPC port | DB package              |
| -------------- | --------- | --------- | ----------------------- |
| `auth-service` | 9001      | 50051     | `@backendworks/auth-db` |
| `auth-worker`  | —         | 50053     | `@backendworks/auth-db` |
| `post-service` | 9002      | 50052     | `@backendworks/post-db` |
| `post-worker`  | —         | 50054     | `@backendworks/post-db` |

### Shared packages and their repositories

| Package                 | npm scope           | Prisma schema covers |
| ----------------------- | ------------------- | -------------------- |
| `@backendworks/auth-db` | `packages/auth-db/` | `User`, `Role`       |
| `@backendworks/post-db` | `packages/post-db/` | `Post`               |

**Auth flow in `post`**: The `AuthJwtAccessGuard` in `post` does **not** use Passport — it calls Auth service via gRPC (`GrpcAuthService.validateToken`) to verify Bearer tokens on every protected request. Authenticated user (`id`, `role`) is attached to `request.user`.

## gRPC Code Generation

Proto files live in `src/protos/`. TypeScript types are **auto-generated** into `src/generated/` — never edit these files manually.

```bash
# Must run before dev or after editing any .proto file
npm run proto:generate   # runs: nestjs-grpc generate --proto ./src/protos --output ./src/generated
npm run dev              # automatically runs proto:generate first
```

- `auth-service` exposes: `ValidateToken` (used by post-service guard)
- `auth-worker` exposes: worker-specific RPCs (email dispatch, async user ops)
- `post-service` exposes: `CreatePost`, `GetPost`, `GetPosts`, `UpdatePost`, `DeletePost`
- `post-worker` exposes: worker-specific RPCs (indexing, async post ops)
- gRPC controllers use `@GrpcController` / `@GrpcMethod` decorators from `nestjs-grpc`

## Developer Workflows

All commands run **per-app** from the app's own directory:

```bash
npm run dev                  # watch mode (proto:generate → nest start --watch)
npm run test                 # unit tests with 100% coverage enforced
```

Shared package commands run from `packages/auth-db/` or `packages/post-db/`:

```bash
npm run prisma:migrate       # dotenv -e .env -- prisma migrate dev
npm run prisma:generate      # regenerates Prisma client from schema
npm run prisma:studio        # opens Prisma Studio
npm run build                # tsc compile → dist/ for publishing
```

Full stack via Docker:

```bash
docker-compose up --build    # from repo root
```

**Database migrations are owned by the packages, not the apps.** Never add a `schema.prisma` inside an app. Always run `prisma migrate dev` from `packages/auth-db/` or `packages/post-db/`.

## Shared Database Packages (`packages/`)

### ORM-Agnostic Design

Each package exposes a repository interface layer. Apps and workers import repositories, never `PrismaClient` directly:

```typescript
// apps import this interface — never PrismaClient
import { IUserRepository, createAuthDbManager } from "@backendworks/auth-db";

// The package wires Prisma internally; swap to TypeORM/Drizzle by replacing
// the implementation only — the interface stays the same
```

### Package structure (same pattern for both)

```
packages/auth-db/
  src/
    client/
      prisma.client.ts             # PrismaClient singleton factory
    repositories/
      user.repository.ts           # Implements IUserRepository using Prisma
    interfaces/
      user.repository.interface.ts # IUserRepository — ORM-agnostic contract
      db-manager.interface.ts      # IAuthDbManager — top-level export shape
    operations/
      base.operations.ts           # Generic findById, findOne, findMany,
                                   #   create, update, softDelete helpers
    index.ts                       # Public API: export interfaces + createAuthDbManager()
  prisma/
    schema.prisma                  # Schema owned here — apps have NO schema.prisma
  package.json                     # name: "@backendworks/auth-db", semver versioned
```

### Versioning rule

- Any **schema change** → bump minor version of the package
- Any **breaking interface change** → bump major version
- Apps and their paired worker must always pin to the **same version range**

## Project-Specific Patterns

### Response Shape

Every HTTP response is wrapped by `ResponseInterceptor` into:

```json
{ "statusCode": 200, "timestamp": "...", "message": "...", "data": { ... } }
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BackendWorks/nestjs-microservices](https://github.com/BackendWorks/nestjs-microservices) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
