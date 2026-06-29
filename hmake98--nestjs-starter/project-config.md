---
trigger: always_on
description: Complete project context embedded below. Do not scan source files to re-derive what is already here.
---

# NestJS Starter — GitHub Copilot Instructions

Complete project context embedded below. Do not scan source files to re-derive what is already here.

## Stack

NestJS 11 · Prisma 7 + `@prisma/adapter-pg` (pg Pool, no binary engine) · PostgreSQL · Redis ioredis · BullMQ · JWT (passport-jwt + argon2) · nestjs-pino · nestjs-i18n · Sentry · Swagger (non-prod only) · TypeScript 6 · Node ≥20

## Directory Layout

```
src/
├── app/
│   ├── app.module.ts              root module
│   ├── config/                    registerAs() factories (index.ts barrel)
│   │   ├── app.config.ts          'app.*'
│   │   ├── auth.config.ts         'auth.accessToken.*' / 'auth.refreshToken.*'
│   │   ├── redis.config.ts        'redis.*'
│   │   ├── doc.config.ts          'doc.*'
│   │   └── seed.config.ts         'seed.admin.*'
│   ├── controllers/health.controller.ts
│   └── enums/app.enum.ts
├── common/
│   ├── common.module.ts           imports all infra; exports DatabaseModule + CacheModule only
│   ├── bullmq/bullmq.module.ts    shared BullMQ Redis connection
│   ├── cache/services/cache.service.ts
│   ├── database/
│   │   ├── database.module.ts     provides+exports DatabaseService, UserRepository
│   │   ├── services/database.service.ts
│   │   ├── repositories/user.repository.ts
│   │   ├── interfaces/user.interface.ts   UserEntity, CreateUserInput, UpdateUserInput
│   │   └── enums/role.enum.ts            re-exports Prisma Role as UserRole
│   ├── doc/decorators/
│   │   └── doc.api-endpoint.decorator.ts  @ApiEndpoint (only decorator for controller methods)
│   ├── request/
│   │   ├── request.module.ts      registers ThrottlerGuard → JwtAccessGuard → RolesGuard
│   │   ├── decorators/auth-user.decorator.ts    @AuthUser()
│   │   ├── decorators/public.decorator.ts       @PublicRoute()
│   │   ├── decorators/roles.decorator.ts        @AllowedRoles([...])
│   │   ├── guards/jwt-access.guard.ts
│   │   ├── guards/jwt-refresh.guard.ts
│   │   └── interfaces/request.interface.ts      IAuthUser = { userId, role }
│   └── response/
│       ├── dtos/response.success.dto.ts    ApiSuccessResponseDto<T>
│       ├── dtos/response.generic.dto.ts    ApiGenericResponseDto
│       ├── filters/response.exception.filter.ts
│       └── interceptors/response.interceptor.ts
├── modules/
│   ├── auth/                      public: login, signup, refresh
│   └── user/                      public: profile CRUD; admin: delete
└── workers/schedulers/            Cron schedulers (MidNightScheduleWorker)
```

## Module Wiring Rules

**AppModule** imports: `ConfigModule.forRoot({ load: configs, isGlobal: true })`, `TerminusModule`, `CommonModule`, `WorkerModule`, `AuthModule`, `UserModule`.

- Feature modules (`AuthModule`, `UserModule`) import `DatabaseModule` directly — never each other.
- `CommonModule` is only imported by `AppModule`.
- New feature module → import `DatabaseModule` + add to `AppModule` imports.
- New repository → add to `DatabaseModule` providers AND exports.

## Guard Order & Auth Decorators

Guards execute in this fixed order (all registered via `APP_GUARD` in `RequestModule`):
1. `ThrottlerGuard` — 10 req / 60s (config: `app.throttle.ttl`, `app.throttle.limit`)
2. `JwtAccessGuard` — JWT validation; bypassed when `@PublicRoute()` metadata present
3. `RolesGuard` — role check; no-op when `@AllowedRoles` metadata absent

```typescript
@PublicRoute()                    // bypass JWT entirely (login, signup, health)
@AllowedRoles([UserRole.ADMIN])  // array required — never spread
@AuthUser()                       // param decorator → IAuthUser = { userId: string, role: UserRole }
@UseGuards(JwtRefreshGuard)       // only on GET /v1/auth/refresh-token
```

**Existing route examples:**
- `POST /v1/auth/login` — `@PublicRoute()` at class level, no auth
- `GET /v1/auth/refresh-token` — `@UseGuards(JwtRefreshGuard)` + `@ApiBearerAuth('refreshToken')`
- `GET /v1/user/profile` — JWT-protected, `@AuthUser()` extracts user
- `DELETE /v1/admin/user/:id` — `@AllowedRoles([UserRole.ADMIN])` at class level
- `GET /health` — `VERSION_NEUTRAL`, `@PublicRoute()`

## Config Factory Pattern

```typescript
// src/app/config/auth.config.ts
export default registerAs('auth', () => ({
    accessToken: { secret: process.env.AUTH_ACCESS_TOKEN_SECRET, tokenExp: process.env.AUTH_ACCESS_TOKEN_EXP },
    refreshToken: { secret: process.env.AUTH_REFRESH_TOKEN_SECRET, tokenExp: process.env.AUTH_REFRESH_TOKEN_EXP },
}));

// In service — ALWAYS getOrThrow, NEVER process.env:
this.configService.getOrThrow<string>('auth.accessToken.secret')
this.configService.getOrThrow<string>('app.http.port')
```

`process.env` is read **only** inside config factories. Everywhere else: `ConfigService.getOrThrow<T>('dot.path')`.

## Repository Pattern

```typescript
@Injectable()
export class PostRepository {
    constructor(private readonly db: DatabaseService) {}

    findById(id: string): Promise<PostEntity | null> {
        return this.db.post.findUnique({ where: { id } });
    }

    async existsById(id: string): Promise<boolean> {
        const found = await this.db.post.findUnique({ where: { id }, select: { id: true } });
        return found !== null;                          // never count(), never findFirst()
    }


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hmake98/nestjs-starter](https://github.com/hmake98/nestjs-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
