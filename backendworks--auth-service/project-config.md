---
trigger: always_on
description: Handles user authentication (JWT), authorization (Passport strategies), and user management. Exposes both a REST API (`:9001`) and a gRPC server (`:50051`) for token validation consumed by the Post service.
---

# Copilot Instructions – Auth Service

## Overview

Handles user authentication (JWT), authorization (Passport strategies), and user management. Exposes both a REST API (`:9001`) and a gRPC server (`:50051`) for token validation consumed by the Post service.

## Developer Workflows

Run all commands from the `auth/` directory:

```bash
npm run dev              # proto:generate → nest start --watch
npm run test             # jest --runInBand, 100% coverage enforced
npm run proto:generate   # regenerates src/generated/auth.ts from src/protos/auth.proto
```

**Database migrations are NOT run from here.** The auth service consumes `@backendworks/auth-db`.
Run all Prisma commands from `packages/auth-db/` instead:

```bash
# from packages/auth-db/
npm run prisma:migrate   # dotenv -e .env -- prisma migrate dev
npm run prisma:studio    # dotenv -e .env -- prisma studio
```

## REST Endpoints

| Method | Path              | Guard                 | Controller                  |
| ------ | ----------------- | --------------------- | --------------------------- |
| POST   | `/auth/login`     | `@PublicRoute()`      | `auth.public.controller.ts` |
| POST   | `/auth/signup`    | `@PublicRoute()`      | `auth.public.controller.ts` |
| GET    | `/auth/refresh`   | `AuthJwtRefreshGuard` | `auth.public.controller.ts` |
| GET    | `/user/profile`   | `@UserAndAdmin()`     | `user.auth.controller.ts`   |
| PUT    | `/user/profile`   | `@UserAndAdmin()`     | `user.auth.controller.ts`   |
| GET    | `/admin/user`     | `@AdminOnly()`        | `user.admin.controller.ts`  |
| DELETE | `/admin/user/:id` | `@AdminOnly()`        | `user.admin.controller.ts`  |

## gRPC Server

Defined in `src/protos/auth.proto`, types auto-generated into `src/generated/auth.ts`.

```protobuf
service AuthService {
  rpc ValidateToken(ValidateTokenRequest) returns (ValidateTokenResponse);
}
```

Controller: `src/app/auth.grpc.controller.ts` — calls `AuthService.verifyToken()` and returns `{ success, payload: { id, role } }`.

## Auth Guards (Passport-based)

Unlike the Post service, this service uses **Passport strategies** for its own HTTP guards:

- `jwt.access.guard.ts` → `AuthJwtAccessStrategy` (validates `Bearer` token, sets `request.user`)
- `jwt.refresh.guard.ts` → `AuthJwtRefreshStrategy` (validates refresh token)
- `roles.guard.ts` → reads `ROLES_DECORATOR_KEY` metadata set by `@AdminOnly()` / `@UserAndAdmin()`

## Key Patterns

### Adding a new endpoint

1. Add DTO in `modules/<domain>/dtos/`
2. Add i18n key in `languages/en/<domain>.json`
3. Decorate controller method with `@MessageKey('domain.success.action', ResponseDto)`
4. Use `@SwaggerResponse(Dto)` / `@SwaggerPaginatedResponse(Dto)` for `@ApiResponse`
5. Add 100% test coverage in `test/unit/<name>.service.spec.ts`

### JWT token flow

- `AuthService.generateTokens()` — signs both access (`15m`) and refresh (`7d`) tokens
- Secrets read from `auth.accessToken.secret` / `auth.refreshToken.secret` config namespaces
- `AuthService.verifyToken()` is the single method called by the gRPC controller

### User soft-delete

`UserAdminService.deleteUser()` sets `deletedAt: new Date()` — never hard-deletes. All queries include `deletedAt: null`.

### Config namespaces

| File              | Namespace | Key example                         |
| ----------------- | --------- | ----------------------------------- |
| `app.config.ts`   | `app.*`   | `app.http.port`, `app.cors.origins` |
| `auth.config.ts`  | `auth.*`  | `auth.accessToken.secret`           |
| `grpc.config.ts`  | `grpc.*`  | `grpc.url`, `grpc.package`          |
| `redis.config.ts` | `redis.*` | `redis.url`, `redis.ttl`            |
| `doc.config.ts`   | `doc.*`   | `doc.enable`, `doc.prefix`          |

## Folder Structure

```
src/
  app/
    app.module.ts               # Wires CommonModule, AuthModule, UserModule, GrpcModule
    app.controller.ts           # GET /health
    auth.grpc.controller.ts     # gRPC: ValidateToken RPC
  common/
    common.module.ts            # Global: config, Joi validation, cache, guards, i18n, interceptors
    config/                     # registerAs() typed factories (app, auth, grpc, redis, doc)
    guards/
      jwt.access.guard.ts       # Passport access token guard
      jwt.refresh.guard.ts      # Passport refresh token guard
      roles.guard.ts            # Role metadata guard
    providers/
      jwt.access.strategy.ts    # PassportStrategy for access tokens
      jwt.refresh.strategy.ts   # PassportStrategy for refresh tokens
    services/
      hash.service.ts           # bcrypt createHash() / match()
      query-builder.service.ts  # findManyWithPagination() — delegates to IUserRepository
    decorators/                 # @PublicRoute, @AdminOnly, @UserAndAdmin, @AuthUser, @MessageKey
    dtos/                       # ApiBaseQueryDto, SwaggerResponse(), SwaggerArrayResponse()
    filters/                    # ResponseExceptionFilter (Sentry on 5xx)
    interceptors/               # ResponseInterceptor — wraps all responses in envelope
    middlewares/                # RequestMiddleware — adds X-Request-ID, structured HTTP logs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BackendWorks/auth-service](https://github.com/BackendWorks/auth-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
