---
trigger: always_on
description: **Global Bible Tools** is a Tanstack Start full-stack application for collaborative Bible translation. It uses:
---

# AGENTS.md — Global Bible Tools Platform

## Project Overview

**Global Bible Tools** is a Tanstack Start full-stack application for collaborative Bible translation. It uses:

- **React 19**, **TypeScript 5** (strict mode)
- **PostgreSQL** via **Kysely** (type-safe query builder) and `pg`
- **Tailwind CSS 4**, **Headless UI**, **Font Awesome** icons
- **use-intl** for i18n (English + Arabic)
- **Zod** for schema validation
- **Vitest** for testing
- **Pino** for structured logging

---

## Commands

### Lint, Format, and Type-Check

```bash
docker compose exec server npm run lint              # ESLint
docker compose exec server npm run format            # Prettier --write on all files
docker compose exec server npm run check-types       # tsc --noEmit (type-check only, no emit)
```

### Testing

```bash
docker compose exec server npm run test              # Vitest in watch mode
docker compose exec server npm run test:run          # Vitest single pass (used in CI)

# Run a single test file
docker compose exec server npx vitest run src/modules/translation/actions/updateGloss.test.ts

# Run all tests under a module directory
docker compose exec server npx vitest run src/modules/translation

# Run tests matching a name pattern
docker compose exec server npx vitest run -t "creates a phrase"

# Run with verbose reporter
docker compose exec server npx vitest run --reporter=verbose src/modules/translation/model/Phrase.unit.ts
```

---

## Architecture

The codebase follows **Domain-Driven Design** layering inside `src/modules/`. Each of the 10 modules (`access`, `bible-core`, `dashboard`, `export`, `languages`, `reporting`, `study`, `translation`, `users`) is structured as:

```
src/modules/<module>/
  actions/        # Next.js Server Actions ("use server") — boundary layer
  use-cases/      # Business logic / application layer
  data-access/    # Repositories — DB reads, domain model mapping
  read-models/    # Query-side read models (Kysely query builders)
  model/          # Domain model classes with domain events
  db/
    schema.ts     # Kysely table interfaces (Generated/Selectable/Insertable)
    migrations/   # SQL migration files
  ui/             # React components specific to this module
  jobs/           # Background job handlers
  test-utils/     # Factories and DB helpers for tests
  __mocks__/      # Vitest module mocks
  index.ts        # Public barrel export
  types.ts        # Shared enums and types within the module
```

**Data flow:** `UI Component` → `Server Fn` → `Use Case` → `Repository / Domain Model`

- Business logic lives in **use cases**.
- DB mapping lives in **repositories**.
- Domain events are emitted by **model classes**.
- Server functions catch errors and call tanstack start primitives (`notFound()`, `redirect()`).

Shared cross-cutting code lives in `src/shared/` (errors, feature-flags, i18n, jobs, ulid).

Path aliases: `@/*` → `src/*`, `@/tests/*` → `tests/*`.

---

## Code Style

### Formatting (Prettier)

- **Double quotes**, **semicolons on**, **2-space indent**, **trailing commas (all)**, **80-char print width**.
- `experimentalTernaries: true` — use the "curious ternary" style:
  ```ts
  const result = condition ? consequentValue : alternateValue;
  ```
- Run `npm run format` before committing, or rely on the Husky pre-commit hook (lint-staged runs Prettier on staged files automatically).

### TypeScript

- **`strict: true`** is enabled. No implicit `any`, strict null checks apply everywhere.
- `@typescript-eslint/no-explicit-any` is **turned off** — explicit `any` is acceptable where practical (e.g., server action signatures, query helpers).
- Prefer **`interface`** over `type` for object shapes. Use `type` for unions, intersections, and computed types.
- Use `as const` objects + a type alias (`type Foo = (typeof FooMap)[keyof typeof FooMap]`) for string-valued enumerations used as discriminated unions.
- Use Kysely's `Generated<T>`, `Selectable<T>`, `Insertable<T>` helpers for DB schema types.
- Use `readonly` on class fields and array return types where mutation is unintended.
- `noImplicitOverride: true` is on — always add the `override` keyword when overriding a base class member.

### Naming Conventions

| Kind                        | Convention                                                 | Example                                 |
| --------------------------- | ---------------------------------------------------------- | --------------------------------------- |
| React component files       | `PascalCase.tsx`                                           | `TranslateWord.tsx`                     |
| Other TypeScript files      | `camelCase.ts` or `kebab-case.ts`                          | `updateGloss.ts`, `form-parser.ts`      |
| Test / unit files           | `<name>.test.ts` / `<name>.unit.ts`                        | `updateGloss.test.ts`, `Phrase.unit.ts` |
| Classes                     | `PascalCase`                                               | `Phrase`, `Policy`                      |
| Interfaces                  | `PascalCase`                                               | `UpdateGlossUseCaseRequest`             |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [globalbibletools/platform](https://github.com/globalbibletools/platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
