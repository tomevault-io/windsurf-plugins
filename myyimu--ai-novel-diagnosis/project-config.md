---
trigger: always_on
description: <!-- one ai-guides:start -->
---

<!-- one ai-guides:start -->
# Codex 工作区 AI 指南

本段内容由 One CLI 基于项目模板为 `AGENTS.md` 自动生成。请优先修改模板 AI 片段，或通过 `one add` 刷新；不要直接手改这段受管内容。

## 工作区

- 根目录：`C:\Users\Legion\Documents\disassemble\ai-novel-first-step`
- AI 提供方：`codex`
- 模板分组数：3

## nestjs-api

适用项目：
- `services/api`

# nestjs-api — Agent Guide

NestJS-based API service. Stack: **NestJS + TypeScript (strict) + class-validator + Drizzle ORM + Jest + structured logging**.

## Project layout

```
src/
├── main.ts                   # bootstrap
├── app.module.ts             # root module
├── core/
│   ├── config/               # ConfigModule + configuration registry
│   ├── exceptions/           # BusinessException + error codes
│   ├── filters/              # HttpExceptionFilter (consistent JSON response)
│   ├── interceptors/         # logging, response shaping
│   └── guards/, pipes/       # cross-cutting concerns
├── modules/<feature>/
│   ├── <feature>.module.ts   # declarative wiring only
│   ├── <feature>.controller.ts # HTTP routing only — ≤ 50 lines per controller
│   ├── <feature>.service.ts    # business logic — stateless singleton
│   ├── dto/                    # DTO classes with class-validator decorators
│   └── *.spec.ts               # unit tests (next to source)
├── dao/repositories/         # The ONLY layer touching Drizzle / DB
└── shared/utils/             # pure helpers
test/                         # *.e2e-spec.ts — real HTTP layer
```

## Architecture boundaries — NEVER violate

- **Controller**: HTTP routing + parameter parsing only. ≤ 50 lines. No business logic, no DB access. Inject services and call them.
- **Service**: All business logic lives here. Stateless, singleton scope. Inject repositories and other services. Never reach into HTTP, never call another controller.
- **Repository** (`src/dao/repositories/`): The only layer that touches Drizzle / the database. Returns typed entities, never raw rows. Never expose ORM types upward.
- **Module** (`src/modules/<feature>/`): Declarative wiring only. No logic in the module file itself.
- Cross-cutting concerns (auth, logging, validation, metrics) belong in **Interceptors / Guards / Pipes / Filters** — never sprinkled across services.

## Engineering discipline — mandatory

These steps are mandatory before declaring any change "done". Not suggestions.

1. `pnpm typecheck` exits 0
2. `pnpm lint` exits 0
3. `pnpm test` passes — and any new code must come with new tests
4. Stage files explicitly with `git add <file>`. Never `git add -A` — it picks up `.env`, `dist/`, debug files.
5. Conventional commit message: `feat(auth): add JWT refresh endpoint`. One commit = one logical change.
6. Never commit secrets. Use `one secrets set <KEY> --env <env>`.
7. Run any destructive operation with `--dry-run` first.

If any fails, stop. Read the failure, fix the underlying problem, then re-run.

## Testing conventions

- Unit tests: `<name>.spec.ts` next to source. Use `Test.createTestingModule` from `@nestjs/testing`.
- Mock external dependencies (DB, HTTP, FS) — never real IO in unit tests.
- **Do NOT mock your own services**. If a test "needs" to mock another service in the same module, the design is wrong — fix the design.
- E2E tests: `test/*.e2e-spec.ts` — real HTTP layer.
- Every new controller endpoint requires: 1 happy path + 1 401/403 (if guarded) + 1 400 (validation error).
- Test names: `should <do something> when <condition>`.
- Coverage target: `src/modules/` 80% branch, `src/dao/` 60%.

## Code style

**DTOs and validation**

- Every external input must be a DTO class (not an inline `{ a: string }`).
- Validate with `class-validator`: `@IsString()`, `@IsEmail()`, `@IsInt() @Min(0)`, etc.
- Output is a DTO or typed response interface — never return raw entities (entities leak `password_hash`, internal FKs, etc.).

**Error handling**

- Domain errors → `BusinessException(code, message, context?)` from `src/core/exceptions/`. Code is machine-readable, message is for humans, context is structured.
- HTTP errors → NestJS built-ins (`NotFoundException`, `BadRequestException`, `UnauthorizedException`).
- `HttpExceptionFilter` normalizes all errors. Trust it — don't write your own response shaping in controllers.
- ❌ NEVER `throw new Error("...")` (no code, no recovery hint).
- ❌ NEVER `try { ... } catch (e) { /* swallow */ }`. Every error either handled meaningfully or rethrown.

**Logging**

- Inject NestJS `Logger` (or pino if configured). NEVER `console.log/error/warn`.
- Structured logs: `logger.log({ user_id, action: 'auth.login' }, 'user logged in')` — not `'user ' + id + ' logged in'`.
- Hot path code: don't log large objects.
- Levels: `debug` for dev, `log/info` for business events, `warn` for recoverable anomalies, `error` for unrecoverable.

**Types**

- TypeScript `strict` mode is on. Keep it.
- ❌ NEVER `any`. Use `unknown` and narrow.
- Public methods must have explicit return types.
- `type` for unions/intersections; `interface` for classes/extension.

**Configuration**

- Inject `ConfigService` from `@nestjs/config`. NEVER `process.env.X` in business code.
- Every env var registered in `src/core/config/configuration.ts` with default + validation.
- Secrets always go through `one secrets`, never raw `.env`.

## Anti-patterns — do not do these

- ❌ SQL strings in services. Use the repository.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [myyimu/ai-novel-diagnosis](https://github.com/myyimu/ai-novel-diagnosis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
