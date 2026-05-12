---
trigger: always_on
description: - **Product name**: PocketWatch
---

# PocketWatch — Project Rules

## Identity & Git

- **Product name**: PocketWatch
- **Dev repo (origin)**: `cryptoxinu/PocketWatcher` (private) — default push target
- **Production repo**: `viperrcrypto/PocketWatch` (public)
- **Branch**: `main` is default
- **Remotes**: `origin` → PocketWatcher (private dev), `production` → PocketWatch (public)

## HARD RULES (never break these)

### File Size Limits
- **Pages**: MAX 400 lines. Extract sections into components.
- **Components**: MAX 300 lines. Split into sub-components.
- **Hooks**: MAX 300 lines per file. Group by feature domain (e.g. `use-finance-budgets.ts`, not one giant `use-finance.ts`).
- **Library files**: MAX 400 lines. Split by concern.
- **API routes**: MAX 200 lines. Extract shared logic to lib/.
- If a file exceeds these limits, SPLIT IT before adding more code.
- NEVER create a "god file" that holds all hooks/components for a module.
- Before adding code to ANY file, check its line count. If adding your code would exceed the limit, split first.

### File Organization
- One hook file per feature domain: `use-portfolio-balances.ts`, `use-portfolio-sync.ts`, etc.
- One component per file. If a page needs 5 sections, that's 5 component files + 1 page file.
- Shared types go in `src/types/`. Shared utils go in `src/lib/`.
- Group by feature, not by type: `portfolio/settings/api-keys-section.tsx`, not `components/sections/api-keys.tsx`.
- Constants, helpers, and types shared across components in a feature go in a `-constants.ts`, `-helpers.ts`, or `-types.ts` file alongside the components.
- Barrel re-exports (`index.ts`) for hook directories only. Components import directly.

### Before Writing Code
- Read the files you'll modify FIRST. Don't guess structure.
- Check existing patterns in the codebase before inventing new ones.
- If you need to add >50 lines to a file, check its current size first. Split if needed.
- Search for existing utilities before writing new ones (`src/lib/`, `src/hooks/`).
- Never duplicate logic — if two components need the same helper, extract it.

### Code Quality
- Immutable data patterns only — NEVER mutate. Return new objects.
- All API routes use `apiError()` from `@/lib/api-error` for error responses.
- All hooks use React Query with the query key factory pattern (see existing hooks).
- TypeScript strict mode — no `any` unless absolutely necessary with a comment explaining why.
- No hardcoded values — use constants or config.
- Functions should be <50 lines. If longer, extract helpers.
- No deep nesting (>4 levels). Use early returns and guard clauses.
- Prefer named exports over default exports (except page.tsx which Next.js requires as default).

### Auth Pattern
- All API routes: `const user = await getCurrentUser(); if (!user) return apiError(..., 401)`
- Use `requireAuth()` or `withAuthEncryption()` from `@/lib/auth`.

### Error Handling
- API routes: always wrap in try/catch, return `apiError()` with unique error code.
- Hooks: let React Query handle errors. Use `onError` callbacks for user-facing toasts.
- Components: use error boundaries for crash recovery, inline error states for data failures.
- NEVER silently swallow errors. Log server-side, toast client-side.

### Performance
- Lazy-load heavy components with `dynamic()` (charts, editors, modals).
- Use `useMemo`/`useCallback` for expensive computations and stable references passed to children.
- Paginate all list endpoints — never return unbounded arrays.
- Use cursor-based pagination for feeds/transactions, offset-based for finite lists.
- Prisma: always `select` or `include` only needed fields. Never `findMany()` without limits.

### Security
- Never expose raw database errors to the client.
- Never store plaintext API keys — use `apiKeyEnc` with server-side encryption.
- Validate all request body fields before database operations.
- Never trust client-side data for authorization — always verify server-side.

### Naming Conventions
- Components: PascalCase (`TransactionFeed.tsx` → `transaction-feed.tsx` file)
- Hooks: `use-{feature}-{domain}.ts` (e.g. `use-portfolio-sync.ts`)
- Lib files: kebab-case (`token-prices.ts`, `pnl-calculator.ts`)
- API routes: kebab-case directories matching the resource (`/api/tracker/api-keys/`)
- Types: PascalCase interfaces, camelCase properties
- Constants: UPPER_SNAKE_CASE for true constants, camelCase for config objects

### What NOT to Do
- Don't add features beyond what was asked — no speculative abstractions.
- Don't add comments to code you didn't write or change.
- Don't create wrapper components for single-use cases.
- Don't add error handling for impossible scenarios.
- Don't refactor surrounding code when fixing a bug — fix the bug only.
- Don't import from barrel files when you can import directly from the source.

## Stack
- **Framework**: Next.js 16 (App Router)
- **Database**: PostgreSQL + Prisma ORM
- **State**: React Query (@tanstack/react-query)
- **Styling**: Tailwind v4, CSS variables for colors (`var(--foreground)`, `var(--card-border)`, etc.)
- **UI**: Material Symbols icons, custom card/button classes (`card`, `btn-primary`, `btn-secondary`, `btn-ghost`)
- **Auth**: Session cookies + bcrypt (local auth, no third-party OAuth)
- **Encryption**: Per-user AES key wrapped with server master key

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [viperrcrypto/PocketWatch](https://github.com/viperrcrypto/PocketWatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
