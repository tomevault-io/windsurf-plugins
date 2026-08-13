---
trigger: always_on
description: Guide for AI agents (and humans) working in this repository. For user-facing documentation see [README.md](./README.md), which remains the high-level source of truth.
---

# AGENTS.md

Guide for AI agents (and humans) working in this repository. For user-facing documentation see [README.md](./README.md), which remains the high-level source of truth.

## What this project is

`nest-hexagonal-boilerplate`: REST API built with **NestJS 10 + strict TypeScript**, **Drizzle ORM** over PostgreSQL (`postgres-js` driver), **JWT auth** (access + refresh) with bcrypt, swappable storage (local/S3), and mail via React Email. Everything follows a **hexagonal architecture** (ports & adapters).

- Package manager: **pnpm 9** (do not use npm or yarn).
- Node: **20** (`.nvmrc`).

## Setup and commands

```bash
cp .env.example .env
pnpm install            # installs husky via `prepare`
docker compose up -d    # PostgreSQL 16 on :5432 (db nest_db)
pnpm db:push            # sync schema (dev)
pnpm start:dev          # API at http://localhost:3000/api, Swagger at /api/docs
```

| Command | Purpose |
|---|---|
| `pnpm build` | Compiles to `dist/` — use it to verify code compiles |
| `pnpm lint` | ESLint with `--fix` |
| `pnpm format` | Prettier over `src/` and `test/` |
| `pnpm test` | Jest (unit tests, `*.spec.ts` under `src/`) |
| `pnpm db:generate` | Generates SQL migration in `drizzle/` from schemas |
| `pnpm db:migrate` / `pnpm db:migrate:run` | Applies migrations (drizzle-kit dev / programmatic CI) |
| `pnpm db:push` | Direct schema sync without files (dev only) |
| `pnpm new:module <singular> [plural]` | Scaffolds a hexagonal module with full CRUD |
| `pnpm email:dev` | Mail template preview on :3001 |

**Verification after a change**: at minimum `pnpm build` and `pnpm lint`. There are no tests written yet (Jest is configured but there are no `*.spec.ts` files nor a `test/` dir); if you add new business logic, write the `*.spec.ts` next to the file.

## Architecture: the 3 hexagonal rules

These rules are **inviolable**. Check them before every edit:

1. **`domain/`** does not import from `application/`, `infrastructure/`, or `@nestjs/*`. Pure TypeScript. (Single deliberate exception: domain exceptions import `HttpStatus` from `@nestjs/common`.)
2. **`application/`** only imports from `domain/` and from `shared/` **ports**. Use cases depend on interfaces, never on implementations.
3. **`infrastructure/`** implements the ports and exposes the use cases (HTTP controllers, Drizzle repositories, service adapters).

The Nest module (`<x>.module.ts`) does the port-to-adapter binding. Swapping DB, storage, or mail = writing another adapter; domain and use cases stay untouched.

## Module anatomy

Every module under `src/modules/<plural>/` follows this shape (see `users/` as the canonical reference):

```
<module>/
├── domain/
│   ├── entities/<x>.entity.ts         # class with invariants, static factory
│   ├── ports/<x>.repository.ts        # interface + Symbol token, same file
│   └── exceptions/<x>.exceptions.ts   # extend DomainException
├── application/
│   ├── dtos/                          # class-validator + @ApiProperty
│   └── use-cases/<verb>-<x>.use-case.ts  # one class, one execute() method
├── infrastructure/
│   ├── http/<x>.controller.ts         # thin controller + Swagger decorators
│   └── persistence/                   # <x>.schema.ts, <x>.mapper.ts, drizzle-<x>.repository.ts
└── <plural>.module.ts                 # bindings { provide: TOKEN, useClass: Adapter }
```

Current modules: `users` (CRUD + DB), `auth` (JWT, no persistence of its own — consumes `USER_REPOSITORY` from `users`), `files` (storage example, no domain layer).

## Code conventions (mandatory)

- **Ports = interface + Symbol token in the same file**: `export const USER_REPOSITORY = Symbol('USER_REPOSITORY')`. Never abstract classes or string tokens.
- **Injection**: `@Inject(TOKEN)` in the constructor + `{ provide: TOKEN, useClass: Adapter }` in the module. Import interfaces as `import { TOKEN, type MyPort }` to avoid dragging domain runtime code.
- **Entities**: immutable, `private constructor`, `static create(props)` factory, business methods return **new instances** (see `User.rename`).
- **Use cases**: one `@Injectable()` class per action, single `execute(...)` method. Named `<verb>-<x>.use-case.ts`.
- **Persistence**: the Drizzle schema lives in `infrastructure/persistence/<x>.schema.ts` (camelCase in TS → snake_case in DB), with types `XRow = typeof x.$inferSelect` and `NewXRow = $inferInsert`. An `XMapper` (`toDomain`/`toPersistence`) separates row from entity. The connection is injected via `@Inject(DATABASE_CONNECTION) db: Database`.
- **Controllers**: thin — validate (pipes), call `useCase.execute()`, and map output with `XResponse.fromDomain(entity)`. Swagger decorators (`@ApiTags`, `@ApiOperation`, `@ApiBearerAuth`) on every endpoint. IDs via `ParseUUIDPipe`.
- **DTOs**: `class-validator` + `@ApiProperty`/`@ApiPropertyOptional`. There is a global `ValidationPipe` with `whitelist: true`, `forbidNonWhitelisted: true`, `transform: true` — do not repeat manual validation.
- **Path aliases**: `@shared/*`, `@modules/*`, `@/*`. Relative paths inside a module; aliases across modules/shared.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MiguelCovarrubiasdaiz/nest-structure](https://github.com/MiguelCovarrubiasdaiz/nest-structure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
