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

- `DATABASE_URL` (Postgres connection URL / required)
- `NEXT_PUBLIC_BETTER_AUTH_URL` (public app URL, e.g., http://localhost:3000)
- `BETTER_AUTH_SECRET` (32-character secret)
- `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`
- `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`
  Note: These are validated strictly via Zod. Missing/invalid values will cause runtime or startup failures.
- For Stripe billing/custom checkout, also set `STRIPE_SECRET_KEY` and `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` (`STRIPE_WEBHOOK_SECRET` is optional for local development).

■ Common Scripts (Bun preferred)

- Dev server: `bun dev` (or `bun run dev` / fallback: `bun run dev`)
- Build: `bun run build` (fallback: `bun run build`)
- Start: `bun run start` (fallback: `bun run start`)
- Lint: `bun run lint` (oxlint)
- Format: `bun run format` (oxfmt)
- Drizzle generate: `bun run db:generate` (create migrations snapshot)
- Drizzle migrate: `bun run db:migrate`
- Drizzle push: `bun run db:push`
- Regenerate better-auth schema: `bun run auth:generate` (uses `bunx` under the hood; overwrites `src/db/schema/auth-schema.ts`)

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

- Server config: `src/lib/auth.ts` (Drizzle adapter + providers).
- Route: `src/app/api/auth/[...all]/route.ts` exports the better-auth handler.
- Client: `src/lib/auth-client.ts` has `baseURL` pinned to `http://localhost:3000`; change only if requested.
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
  3. Ensure all translation files have the same keys.
- Use `next-intl` for translations in components (e.g., `useExtracted()` hook).
- Do not branch UI copy on locale with flags like `isJapanese` or `locale === "ja"` when `useExtracted()` can express it. Prefer translated strings and structure the JSX so ordering and wording come from translations, not locale conditionals.
- Before completing i18n-related changes, verify that all language files are in sync.

■ Directory Guidelines

- Pages/Layouts: `src/app/**`
- Shared components: `src/components/**`
- UI (generated): `src/components/ui/**`
- DB/ORM: `src/db/**`
- Auth/Client libs: `src/lib/**`
- Custom hooks: `src/hooks/**`

■ Prohibited/Use Caution

- Do not add heavyweight dependencies or change the toolchain (e.g., new formatter) without explicit approval.
- Avoid unnecessary renames of files/exports. Keep diffs minimal and targeted.
- Do not add license/copyright headers.
- Avoid destructive edits to generated files (especially `src/components/ui/*`). If required, justify and document impact.
- Never hardcode secrets. Use environment variables.

■ Validating Changes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [teamzisty/deni-ai](https://github.com/teamzisty/deni-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
