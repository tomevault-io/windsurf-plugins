---
trigger: always_on
description: AGENTS.md (Agent Working Guide for deni-ai)
---

AGENTS.md (Agent Working Guide for deni-ai)

This file applies to the entire repository tree rooted here. Follow these rules, steps, and cautions when making changes. If a deeper directory contains its own AGENTS.md, the more specific one takes precedence. Direct instructions from system/developers/users override this file.

■ Project Overview

- Framework: Next.js App Router (next@16 canary, React 19, React Compiler enabled)
- Language/Types: TypeScript (strict)
- Runtime: Bun (preferred; bun.lock present) or Node.js 20+
- Lint/Format: oxlint (linting) and oxfmt (formatting)
- Styles: Tailwind CSS v4 (via `@tailwindcss/postcss`)
- UI: shadcn/ui (generated under `src/components/ui/*`)
- DB: Postgres (Neon serverless) + Drizzle ORM (`drizzle-kit`)
- Auth: better-auth (Drizzle adapter)

■ Required Environment Variables (as enforced by `src/env.ts`)

Source of truth: `src/env.ts`. Starter: `.env.example`. Human setup guide: `SETUP.md`.
Empty optional strings are treated as unset (`emptyStringAsUndefined`) for Docker/Dokploy.

Required (Zod will fail startup/build without them):

- `DATABASE_URL` (Postgres / Neon)
- `NEXT_PUBLIC_BETTER_AUTH_URL` (public app URL, e.g. http://localhost:3000)
- `BETTER_AUTH_SECRET` (exactly 32 characters)
- `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`
- `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`
- `STRIPE_SECRET_KEY` (always required by validation; use `NEXT_PUBLIC_BILLING_DISABLED` to hide billing UI)
- `GOOGLE_GENERATIVE_AI_API_KEY`, `ANTHROPIC_API_KEY`, `GROQ_API_KEY`, `OPENROUTER_API_KEY`
- `BRAVE_SEARCH_API_KEY`
- `TURNSTILE_SECRET_KEY`, `NEXT_PUBLIC_TURNSTILE_SITE_KEY`

Optional:

- Stripe: `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`, `STRIPE_WEBHOOK_SECRET`, `STRIPE_FLASH_OFFER_COUPON_ID`
- voids.top: `VOIDS_MODE=true|1` routes platform OpenAI + Anthropic via voids; when enabled `VOIDS_API_KEY` is required; optional `VOIDS_BASE_URL`
- Email (Cloudflare Email Sending): `CLOUDFLARE_ACCOUNT_ID`, `CLOUDFLARE_API_TOKEN`
- Blog admin: `BLOG_ADMIN_EMAILS` (falls back to `AFFILIATE_ADMIN_EMAILS`)
- Blog admin: `BLOG_ADMIN_EMAILS` (falls back to `AFFILIATE_ADMIN_EMAILS`)
- Rate limit: `UPSTASH_REDIS_REST_URL` / `UPSTASH_REDIS_REST_TOKEN` or `KV_REST_API_URL` / `KV_REST_API_TOKEN`
- Uploads: `UPLOADTHING_TOKEN`
- Client: `NEXT_PUBLIC_BILLING_DISABLED`, AdSense (`NEXT_PUBLIC_ADSENSE_*`)

■ Common Scripts (Bun preferred)

- Dev server: `bun dev`
- Build: `bun run build` (runs `typecheck` then `next build`)
- Start: `bun start` / `bun run start`
- Lint: `bun run lint` (oxlint); fix: `bun run lint:fix`
- Format: `bun run format` (oxfmt)
- Typecheck: `bun run typecheck`
- Drizzle generate: `bun run db:generate`
- Drizzle migrate: `bun run db:migrate` (`.env.production`); local: `bun run db:migrate:dev` (`.env.local`)
- Drizzle push: `bun run db:push`
- Regenerate better-auth schema: `bun run auth:generate` (overwrites `src/db/schema/auth-schema.ts`)
- Disposable email list: `bun run disposable:refresh`
- Tools: `bun run tools:commit`, `bun run tools:codename`
- React doctor: `bun run doctor`

■ Coding Conventions

- Formatting/Linting: Adhere to oxlint and oxfmt. Run `bun run lint` and `bun run format` before submitting changes.
- Exports: Prefer named exports where reasonable. Match existing code style.
- Type safety: Keep TypeScript strict. Avoid `any`; if unavoidable, scope it narrowly.
- Module paths: Use the `@/*` alias (from `tsconfig.json`) to avoid deep relative paths.
- File naming: Follow existing kebab-case for files (e.g., `auth-client.ts`).
- React/Next: App Router patterns (`src/app/**/page.tsx`, `layout.tsx`). Respect server/client component boundaries.
- React Compiler: Avoid sharing mutable closures or side effects that break assumptions. Follow existing patterns.

■ Database & Migrations (Drizzle)

- Schema files live in `src/db/schema/*`; aggregated exports in `src/db/schema/index.ts`.
- Migrations are output to `migrations/` (see `drizzle.config.ts`).
- Driver: Neon (`drizzle-orm/neon-http`). `DATABASE_URL` must be set.
- Typical flow:
  1. Edit schema → 2) `bun run db:generate` → 3) `bun run db:migrate` or `bun run db:push`
- Caution: For destructive changes (dropping columns, type changes), plan safe migrations and backups.

■ Authentication (better-auth)

- Server config: `src/lib/auth.ts` (Drizzle adapter + Google/GitHub OAuth, magic link, anonymous, passkey, 2FA).
- Route: `src/app/api/auth/[...all]/route.ts` exports the better-auth handler.
- Client: `src/lib/auth-client.ts` — do not change baseURL unless explicitly requested.
- Regenerate schema with `bun run auth:generate` (may overwrite `auth-schema.ts`).

■ Frontend & Styles

- Tailwind v4; keep a utility-first approach.
- shadcn/ui generated files under `src/components/ui/*` are generally not edited and are excluded from linting.
  - If changes are absolutely necessary, keep them minimal and non-breaking to component APIs.
- Shared utilities belong in `src/lib/utils.ts`; reusable logic goes under `src/lib/`.

■ Internationalization (i18n)

- Translation files are located in `messages/` directory (`en.json`, `ja.json`, etc.).
- When adding new user-facing text strings:
  1. Add the key and English text to `messages/en.json`.
  2. Add the corresponding Japanese translation to `messages/ja.json`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deniaiapp/app](https://github.com/deniaiapp/app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
