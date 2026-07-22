---
trigger: always_on
description: pnpm dev           # Start dev server with Turbopack
---

# AGENTS.md - Coding Agent Guidelines for SplitPro

## Build/Lint/Test Commands

### Development

```bash
pnpm dev           # Start dev server with Turbopack
pnpm d             # Full dev setup (install, docker, migrate, dev)
pnpm dx            # Setup dependencies (install, docker up, migrate dev)
pnpm dx:up         # Start Docker containers
pnpm dx:down       # Stop Docker containers
```

### Worktrees

- when using git worktrees, store them in the .worktrees directory in the root of the project folder
- when creating a worktree, copy `.env` from the root project into the worktree
- update worktree-local app urls/port to avoid conflicts with the main workspace:
  - set `NEXTAUTH_URL` and `NEXTAUTH_URL_INTERNAL` to a unique localhost port
  - run dev server with that same port (for example `pnpm dev --port 3174`)
- use a separate postgres instance per worktree (different host port + db name)
  - set unique values in the worktree `.env`: `POSTGRES_PORT`, `POSTGRES_DB`, `POSTGRES_CONTAINER_NAME`, `DATABASE_URL`
  - start dedicated db container, for example:
    - `docker run -d --name <unique-container-name> -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=strong-password -e POSTGRES_DB=<worktree-db> -p <worktree-port>:5432 ossapps/postgres:17.7-trixie postgres -c shared_preload_libraries=pg_cron -c cron.database_name=<worktree-db> -c cron.timezone=UTC`
  - enable pg_cron extension once: `psql "postgresql://postgres:strong-password@localhost:<worktree-port>/<worktree-db>" -c "CREATE EXTENSION IF NOT EXISTS pg_cron;"`
  - initialize schema and data: `pnpm exec prisma migrate reset --force --skip-seed && pnpm db:seed`

### Database

```bash
pnpm db:push       # Push Prisma schema to database
pnpm db:studio     # Open Prisma Studio
pnpm db:dev        # Run Prisma migrations (dev)
pnpm db:seed       # Seed the database
pnpm generate      # Generate Prisma client
```

### Testing

```bash
pnpm test                              # Run all tests
pnpm test:watch                        # Run tests in watch mode
pnpm test src/tests/simplify.test.ts   # Run a specific test file
pnpm test -- -t "test name pattern"    # Run tests matching pattern
```

### Linting & Formatting

```bash
pnpm lint                    # Run oxlint with type-aware checking
pnpm prettier --write .      # Format all files
pnpm prettier --check .      # Check formatting without changes
pnpm tsgo --noEmit           # Type check (used in pre-commit)
```

### Building

```bash
pnpm build         # Build for production
```

### CI Pipeline (runs on PRs)

1. `pnpm prettier --check .` - Format check
2. `pnpm lint` - Linting
3. `pnpm tsgo --noEmit` - Type checking
4. `pnpm test` - Unit tests
5. `pnpm build --no-lint` - Build verification

## Code Style Guidelines

### Formatting (Prettier)

- Semicolons: required
- Quotes: single quotes
- Trailing commas: all
- Print width: 100 characters
- Tailwind CSS classes are auto-sorted

### TypeScript Configuration

- Strict mode enabled
- Target: ES2022
- `noUncheckedIndexedAccess: true` - handle undefined for index access
- Path aliases:
  - `~/*` maps to `./src/*`
  - `@/*` maps to `./*`

### Import Organization

1. External packages first (react, next, third-party libs)
2. Internal imports using `~/` alias
3. Relative imports for nearby files
4. Use `type` keyword for type-only imports

### Environment Variables

- Server-side vars: Use `env.VAR_NAME` (available only on server).
- Client-side vars: Use `env.NEXT_PUBLIC_VAR_NAME` (exposed to browser, prefixed with `NEXT_PUBLIC_`).
- Defaults: Hardcode in `src/env.ts` using Zod `.default()` for fallback values.

```typescript
import { HeartHandshakeIcon, X } from 'lucide-react';
import { useTranslation } from 'next-i18next';
import React, { useCallback } from 'react';

import { type CurrencyCode } from '~/lib/currency';
import { useAddExpenseStore } from '~/store/addStore';
import { api } from '~/utils/api';

import { Button } from '../ui/button';
```

### Naming Conventions

- **Components**: PascalCase (`AddExpensePage.tsx`, `Button.tsx`)
- **Utility files**: camelCase (`utils.ts`, `currency.ts`)
- **Test files**: `*.test.ts` pattern
- **Pages**: kebab-case for routes (`import-splitwise.tsx`)
- **Dynamic routes**: `[paramName].tsx`
- **Functions**: camelCase (`calculateParticipantSplit`)
- **Constants**: UPPER_SNAKE_CASE (`DEFAULT_CATEGORY`)
- **Types/Interfaces**: PascalCase (`AddExpenseState`)
- **Hooks**: camelCase with `use` prefix (`useAddExpenseStore`)

### Yoda Conditions

Use literal on the left side for comparisons:

```typescript
if ('authenticated' === status) { ... }
if (0n === amount) { ... }
if ('' === description) { ... }
```

### BigInt Usage

- **All monetary values are BigInt** (cents/smallest unit) to prevent rounding errors
- Use `n` suffix for BigInt literals: `0n`, `1n`, `10000n`
- Use `BigMath` helper (`src/utils/numbers.ts`) for arithmetic: `abs`, `sign`, `min`, `max`, `roundDiv`
- Use `getCurrencyHelpers({ currency, locale })` for display/parsing

### Error Handling

**tRPC API errors:**

```typescript
if (!group) {
  throw new TRPCError({ code: 'NOT_FOUND', message: 'Group not found' });
}
```

**Client-side errors:**

```typescript
try {
  await mutation.mutateAsync(data);
} catch (error) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oss-apps/split-pro](https://github.com/oss-apps/split-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
