---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A NestJS CRUD monorepo (`@nestjs-crud/core`) that auto-generates RESTful CRUD endpoints for NestJS controllers. Supports TypeORM, Drizzle, MikroORM, and Prisma. Managed with Yarn workspaces + Lerna; builds use TypeScript composite project references (`tsc -b`).

## Packages

Authoritative list lives at `ls packages/` and `lerna.json`'s `packages` glob — do not trust hardcoded counts in docs. Current dependency chain:

```
util → request → core → typeorm
                      → drizzle
                      → mikro-orm
                      → prisma
```

- **`@nestjs-crud/util`** — Tiny type-check utilities (`isNil`, `isArrayFull`, etc.)
- **`@nestjs-crud/request`** — `RequestQueryBuilder` (frontend query construction) and `RequestQueryParser` (backend query parsing). Handles search conditions, filters, joins, sorting, pagination
- **`@nestjs-crud/core`** — Core framework: `@Crud()` decorator, `CrudRoutesFactory`, `CrudRequestInterceptor`, `CrudResponseInterceptor`, `@CrudAuth()`, `@Override()`, `@ParsedRequest()`, `CrudConfigService`, `CrudCacheNotConfiguredError`
- **`@nestjs-crud/typeorm`** — `TypeOrmCrudService<T>` — translates parsed requests into `SelectQueryBuilder` queries
- **`@nestjs-crud/drizzle`** — `DrizzleCrudService<T>` — translates parsed requests into Drizzle query builder operations
- **`@nestjs-crud/mikro-orm`** — `MikroOrmCrudService<T>` — translates parsed requests into EntityManager operations
- **`@nestjs-crud/prisma`** — `PrismaCrudService<T>` — translates parsed requests into PrismaClient operations

## Build Commands

```bash
yarn build          # Build all packages (native tsc -b; composite project references walk dep order)
yarn clean          # Remove lib/ dirs and *.tsbuildinfo files
yarn rebuild        # clean + build
yarn lint           # ESLint with --fix on all package .ts files
yarn format         # Prettier via pretty-quick
```

**`yarn rebuild` shadow note.** Yarn 4 ships a built-in `yarn rebuild` that shadows the project script; bare `yarn rebuild` runs postinstall scripts, NOT clean+build. Use `yarn run rebuild` or `yarn clean && yarn build`.

TypeScript uses composite project references. Each package compiles `src/` → `lib/`. Path aliases (`@nestjs-crud/*` → `packages/*/src`) are configured in root `tsconfig.json`.

## Testing

Jest 30 + ts-jest + jest-extended. Tests resolve `@nestjs-crud/*` imports directly to source via `moduleNameMapper` (no build needed).

```bash
# Run a single test file via root config (core/request/util specs only)
npx jest packages/core/test/crud.decorator.base.spec.ts

# Run tests matching a name pattern
npx jest --testNamePattern="getManyBase"

# Per-adapter integration tests (must use the scoped scripts — see rule below)
yarn test:typeorm:postgres   yarn test:typeorm:mysql
yarn test:drizzle:postgres   yarn test:drizzle:mysql
yarn test:mikro-orm:postgres yarn test:mikro-orm:mysql
yarn test:prisma:postgres    yarn test:prisma:mysql

# Aggregator (parity + all 8 adapter cells)
yarn test:all
yarn test:parity   # cross-adapter parity specs only
yarn test:coverage # coverage report
```

**Per-adapter test scoping rule.** Each adapter has its own `packages/<adapter>/jest.config.js` with `testMatch` scoped to that package's `test/`. Adapter integration runs MUST go through `yarn test:<adapter>:<db>` — those scripts invoke `jest --config packages/<adapter>/jest.config.js`, NOT root `yarn test`. Running root `yarn test` against an adapter package mixes ESM/CJS specs and pulls in non-target adapter specs (the failure mode that triggered the per-package configs in the first place). Coverage thresholds are enforced per-package via `coverageThreshold` blocks.

**MikroORM ESM caveat.** `yarn test:mikro-orm` (and `:postgres`/`:mysql` variants) is the ONLY supported way to run `packages/mikro-orm/test/*.spec.ts`. `@mikro-orm/core` v7 is pure ESM (`import.meta.url`); the script sets `NODE_OPTIONS=--experimental-vm-modules` and points Jest at `packages/mikro-orm/jest.config.js` (ts-jest ESM preset). Invoking `npx jest packages/mikro-orm/test/...` directly will fail with `SyntaxError: Cannot use 'import.meta' outside a module`.

**MikroORM seed CLIs use `tsx`, not `ts-node --esm`.** `db:prepare:mikro-orm:*` runs via `npx tsx` for ESM-native `.ts` execution without the `NODE_OPTIONS` dance. Don't switch the seed CLIs back to `ts-node --esm` — the jest test runs (which DO need `--experimental-vm-modules`) and the seed CLIs (which don't) are intentionally separate.

### Test categories

- **`packages/core/test/`** — Unit tests for decorators, interceptors, config service. No database needed.
- **`packages/request/test/`** — Unit tests for query builder/parser. No database needed.
- **`packages/typeorm/test/`** — Integration tests requiring a live database. Tests use `packages/typeorm/test/__fixture__/app/` as the fixture — a self-contained NestJS app (entities, modules, services, seeds, ORM configs) imported directly by the spec files.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kodjunkie/nestjs-crud](https://github.com/kodjunkie/nestjs-crud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
