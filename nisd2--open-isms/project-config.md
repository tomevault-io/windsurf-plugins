---
trigger: always_on
description: bun install              # Install dependencies (ALWAYS use bun, never npm/yarn)
---

# CLAUDE.md — NIS2 Compliance Platform

## Quick Reference

```bash
bun install              # Install dependencies (ALWAYS use bun, never npm/yarn)
bun run dev              # Dev server on port 3026 (webpack, not turbopack)
bun run typecheck        # tsc --noEmit — must pass with ZERO errors
bun run build            # Production build — must pass clean
bun db:generate          # Generate Drizzle migration after SCHEMA changes
bun db:framework-migration # Generate data migration after FRAMEWORK DATA changes
bun db:migrate           # Apply migrations
bun db:seed              # Seed database (drops + recreates dev data)
```

## Hard Rules

- **Zero type errors** — always verify with `bun run typecheck` before finishing work
- **No `as any` casts** — use proper types, generics, or type narrowing
- **No non-null assertions (`!`)** — use explicit null checks, early returns, or narrowing. If a value is guaranteed non-null by prior logic, add a runtime check
- **NEVER `drizzle-kit push`** — ALWAYS `bun db:generate` then `bun db:migrate`. Push bypasses migration tracking and causes DB state drift. No exceptions.
- **NEVER destroy production data in a migration or a deploy-time seed.** This is OpenGRC: a real open-source GRC platform holding customers' compliance evidence, sign-offs, and progress (`companyRequirementStatus`, `evidence`, `requirementAssignment`). Migrations TRANSFORM, they do not lose: add the new shape, backfill from the old, then drop the old. Data is destroyed only with explicit, intentional, reviewed cause. Changes to reference/framework data (legal refs, CIR points, priorities, frequencies, satisfaction pairs) ship as a proper, idempotent migration that touches reference rows only — run `bun db:framework-migration`, which regenerates the sync from `packages/grc-data-model` and appends the journal entry. `bun db:generate` will NOT do this: drizzle-kit diffs the schema, so a row-content change produces an empty migration and the edit silently never reaches an already-seeded database. A seed that runs on deploy must never delete customer-scoped rows. If you are unsure whether a row is customer data or reference data, treat it as customer data.
- **No "wizard" or "demo"** in route names or visible UI
- **KISS** — don't over-engineer. Simple > clever. 3 similar lines > premature abstraction
- **Turbopack is the default for `next build` since 16.2** — the prior "Webpack required" rule (TW v4 PostCSS conflict) was fixed mid-16.x. `bun run build` uses Turbopack; `bun run build:webpack` is the escape hatch. Dev still uses webpack via `bun run dev` (HMR is steadier today); `bun run dev:turbo` is opt-in.
- **Bun only** — package manager is bun, not npm or yarn

## Tech Stack

| Layer | Tech | Version | Notes |
|-------|------|---------|-------|
| Framework | Next.js | 16.1.6 | App Router, SSR-first |
| React | React | 19.1.0 | `useOptimistic`, `useTransition` available |
| TypeScript | TS | 6.0.3 | Strict mode. Note: 6.x errors on deprecated options such as `baseUrl` (TS5101) |
| Styling | Tailwind CSS | 4.1.0 | `@theme inline` for shadcn color vars |
| Validation | Zod | **4.x** | NOT v3. Uses `_def.type` not `_def.typeName` |
| ORM | Drizzle | 0.45.x | PostgreSQL, relational queries |
| API | tRPC | 11.1.0 | superjson transformer |
| Auth | NextAuth | 5.0.0-beta.32 | JWT strategy, Google OAuth |
| i18n | next-intl | 4.8.2 | Cookie-based locale, DE default + EN + NL |
| AI | Vercel AI SDK | 6.x | + @ai-sdk/xai (grok-2-1212) for form prefill |
| Storage | AWS S3 | SDK v3 | Presigned URLs for evidence uploads |
| Email | Resend | — | Transactional email delivery |

## Architecture

### SSR-First Pattern
Server components fetch data via `api` (tRPC server caller), pass serializable props to client components. Client components use `"use client"` directive.

```
Server Page → api.router.procedure() → Client Component (props)
```

After mutations: `router.refresh()` re-fetches server component data.

### Unified Form Pipeline
ALL compliance requirement forms use the same pipeline. No hand-built forms:

```
DB requirement_form_field → buildDynamicSchema() → SchemaForm → shadcn components
```

File-type fields automatically render the `FileUpload` component (S3 presigned URL upload).

### tRPC Procedures
- `publicProcedure` — no auth
- `protectedProcedure` — requires `ctx.userId`, auto-logs all mutations to audit trail
- `adminProcedure` — requires role === "admin"
- `reviewerProcedure` — requires role in ["admin", "reviewer", "legal_reviewer"]

Auto-audit middleware extracts entity ID from input (checks: id, statusId, evidenceId, etc.) and logs after mutation completes (fire-and-forget).

### Auth
- **Dev**: auto-injects seed user via `lib/auth/dev-user.ts`
- **Prod**: Google OAuth, middleware redirects to `/auth/signin`
- Session: `getSession()` from `lib/auth`

### Database
- PostgreSQL via `DATABASE_URL` env var
- **GRC-core schema lives in `@nisd2/grc-data-model`** (workspace package at `packages/grc-data-model/`, mirrored to `github.com/NISD2/grc-data-model` via `git subtree push`)
  - Drizzle tables: `framework`, `requirement`, `requirement-satisfaction`, `supplier`, `asset`, `risk`, `incident`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NISD2/open-isms](https://github.com/NISD2/open-isms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
