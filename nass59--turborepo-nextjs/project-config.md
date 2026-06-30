---
trigger: always_on
description: You are an expert AI pair‑programmer for this Turborepo (Next.js 15 + React 19 + Tailwind 4). Prefer precision, project conventions, and minimal diff changes.
---

You are an expert AI pair‑programmer for this Turborepo (Next.js 15 + React 19 + Tailwind 4). Prefer precision, project conventions, and minimal diff changes.

## 1. Architecture & Boundaries
Apps: `apps/web` (App Router) & `apps/storybook` (component docs). Shared UI & utilities live in `packages/design-system` (exports via path sub-entries) and TS configs in `packages/typescript-config`. Feature code in `apps/web` is grouped by route segment (e.g. `app/(home)`) and domain modules under `modules/` (e.g. `home/`, `projects/`, `messages/`)—keep business logic out of leaf pages. Auth enforced centrally via `apps/web/middleware.ts` (Clerk route guard using `createRouteMatcher` + `auth.protect()`) – don't duplicate per-route auth.

**API Layer:** tRPC for type-safe client/server communication (see `apps/web/trpc/`). Routers defined in `modules/*/server/procedures.ts`, consumed via `useTRPC()` hook in client components. All procedures use `superjson` transformer. Protected routes use `protectedProcedure` with Clerk auth middleware.

**Background Jobs:** Inngest for async processing (`apps/web/inngest/`). Client initialized with app ID `vibe-development`. Functions defined in `inngest/functions.ts`.

## 2. Modules Directory Conventions

Hard rule: NO BARREL FILES (no `index.ts` that aggregate exports). Each import path must point directly to the needed file to:
- Avoid unintended bundle bloat from tree-shaking edge cases.
- Preserve clear client/server component boundaries in Next.js / React 19.
- Simplify static analysis and dead-code elimination.

Structure per module (example: `home/`):
- `ui/` Presentational components (server-first). Add `'use client'` only when interaction or browser APIs required.
- `server/` Server-only functions (queries, mutations). Return serializable, typed data.
- `lib/` Pure utilities (formatters, mappers). No React.
- `hooks/` Client hooks (state, effects) when unavoidable.
- `types.ts` Public types for consumers.
- `constants.ts` Stable configuration values / string literals.
- `README.md` Module-specific notes.
- Tests: colocate (`*.test.ts` / `*.test.tsx`).

Guidelines:
1. Pages import only what they need: `import { Header } from '@/modules/home/ui/home-header'`.
2. Never re-export a client component through a server file (prevents boundary confusion).
3. Keep server functions narrow: one responsibility, clear name (`getHomeContent`).
4. Prefer data shaping inside `server/` before passing into `ui` components.
5. Avoid cross-module deep imports; if repetition emerges, elevate shared logic to a dedicated module or shared package.
6. Keep Tailwind classes sorted (Biome) & avoid inline logic inside JSX where a small helper improves clarity.

Rationale for no barrels:
- Barrel patterns can accidentally pull in client-only code into server bundles (and vice versa) due to eager evaluation.
- Explicit imports make bundle inspection & perf audits easier.
- Encourages smaller, intention-revealing files and discourages god-modules.

If a future need for grouped exports emerges (e.g., IDE ergonomics), prefer generating typed import snippets or using path aliases over barrels.

## 3. Data, Env, Auth
**Env:** Schema validated in `apps/web/env.mjs` with `@t3-oss/env-nextjs` + zod; always import from `@/env` instead of `process.env`. Server vars: `CLERK_SECRET_KEY`, `DATABASE_URL`, `OPENAI_API_KEY`, `E2B_API_KEY`, `GITHUB_ACCESS_TOKEN`. Client vars (prefixed `NEXT_PUBLIC_`): Clerk keys, `APP_URL`, Cloudinary. When adding new env vars: (1) add to `env.mjs` schema + `runtimeEnv`, (2) if build-time required, add to `turbo.json` `build.env` array.

**Database:** PostgreSQL + Prisma (ADR 0005). Hosted on NeonDB (serverless Postgres). Schema in `apps/web/prisma/schema.prisma`. Connection string from `env.DATABASE_URL`. Use `prisma generate` to update client, `prisma db push` for schema sync, `prisma migrate dev` for migrations.

**Auth:** Clerk (ADR 0006) with GitHub OAuth. Server-side: use `auth()` from `@clerk/nextjs/server` (returns `{ userId }`). Client: Clerk hooks in `'use client'` components. Middleware in `apps/web/middleware.ts` protects non-public routes; public routes defined via `createRouteMatcher(['/'])`. tRPC context automatically includes `clerkUserId`.

## 4. Rendering Patterns
Default to Server Components. Add `'use client'` only for interactivity (forms, animations, theme toggles, Zustand stores). Co-locate interactive child component instead of promoting page to client. Example: `app/(public)/(home)/page.tsx` stays server; interactive buttons come from `modules/home/ui/*` (client if needed). MDX pipeline configured in `next.config.ts` + custom mapping in `mdx-components.tsx`—extend there, not ad hoc in pages.

## 5. Styling & Design System
Import primitives from `@workspace/design-system/components/*` (e.g., `@workspace/design-system/components/ui/button`), utilities from `@workspace/design-system/lib/utils` (`cn`). Design system uses Radix UI primitives + Tailwind 4. Prefer existing tokens/utilities; add new variants via class-variance-authority patterns used in DS components.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nass59/turborepo-nextjs](https://github.com/nass59/turborepo-nextjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
