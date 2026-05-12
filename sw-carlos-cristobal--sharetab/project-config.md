---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

ShareTab — open-source, self-hosted Splitwise alternative with AI receipt scanning. Targets Unraid (Docker).

## Tech Stack

- **Framework:** Next.js 16 (App Router) + TypeScript
- **API:** tRPC v11 (end-to-end type-safe)
- **ORM:** Prisma 7 + PostgreSQL 16 (via `@prisma/adapter-pg`)
- **Auth:** NextAuth v5 (email/password + OAuth)
- **UI:** TailwindCSS 4 + shadcn/ui (v4, uses `@base-ui/react` — use `render` prop instead of `asChild`) + next-themes (dark mode)
- **AI:** Pluggable providers (OpenAI, OpenAI-Codex, Claude, Meridian, Ollama, OCR fallback) via `src/server/ai/`
- **i18n:** next-intl (9 locales: en, es, sv, fr, de, pt-BR, ja, zh-CN, ko)

## Commands

```bash
npm run dev          # Start dev server (turbopack)
npm run dev:full     # Start embedded PostgreSQL + dev server (all-in-one)
npm run build        # Production build
npm run start        # Start production server
npm run lint         # ESLint
npm test             # Run unit tests (Vitest)
npm run test:watch   # Unit tests in watch mode
npm run test:e2e     # Run Playwright e2e tests
npm run lint:i18n    # Check translations for missing/extra keys
npx prisma generate  # Regenerate Prisma client after schema changes
npx prisma db push   # Push schema without migration (dev only)
```

## Architecture

- `src/server/` — Backend: auth config, Prisma client, tRPC routers, AI providers, pure calculation libs
- `src/server/db.ts` — Prisma client singleton (uses `@prisma/adapter-pg` with `PrismaPg`)
- `src/server/auth.ts` — NextAuth v5 config (Credentials + optional Google OAuth + optional Nodemailer magic link)
- `src/server/trpc/init.ts` — tRPC context, `publicProcedure`, `protectedProcedure`, `groupMemberProcedure`
- `src/server/trpc/router.ts` — Root app router (exports `AppRouter` type)
- `src/server/trpc/routers/` — Individual routers: auth, groups, expenses, balances, settlements, activity, receipts, guest, admin
- `src/server/lib/balance-calculator.ts` — Pure functions for debt simplification and balance computation (extracted for testability)
- `src/app/[locale]/` — Next.js App Router pages under i18n locale segment. `(auth)/` for login/register, `(app)/` for authenticated pages
- `src/components/` — React components organized by domain
- `src/components/providers.tsx` — Client-side tRPC + React Query + SessionProvider + ThemeProvider wrapper
- `src/lib/trpc.ts` — Client-side tRPC React hooks
- `src/lib/utils.ts` — `cn()` utility for Tailwind class merging
- `src/generated/prisma/` — Auto-generated Prisma client (do not edit, gitignored)
- `prisma/schema.prisma` — Database schema (money stored as Int cents)
- `prisma.config.ts` — Prisma v7 config (datasource URL lives here, not in schema.prisma)
- `src/i18n/routing.ts` — Locale list, default locale, and next-intl routing config
- `src/i18n/request.ts` — Server-side locale resolution for next-intl
- `src/i18n/navigation.ts` — Locale-aware `Link`, `redirect`, `usePathname`, `useRouter`
- `messages/{locale}/` — Translation files with namespaces: admin, auth, common, dashboard, expenses, groups, settings
- `docker/` — Dockerfile (multi-stage) + docker-compose.yml

## Key Conventions

- All monetary amounts are stored as integers in cents (e.g., $12.99 = 1299)
- tRPC routers live in `src/server/trpc/routers/`
- `protectedProcedure` requires auth; `groupMemberProcedure` requires group membership
- AI providers implement the `AIProvider` interface in `src/server/ai/provider.ts`
- `src/middleware.ts` — NextAuth middleware protecting authenticated routes
- `prisma/seed.ts` — Demo data seed script (run with `npm run db:seed`); idempotent — skips if data already exists
- Prisma v7: datasource URL is configured in `prisma.config.ts`, not in `schema.prisma`
- Prisma v7: PrismaClient requires `@prisma/adapter-pg` adapter in constructor
- Prisma v7: import from `@/generated/prisma/client` (not `@/generated/prisma` — no index.ts)
- shadcn/ui v4: Button uses `render` prop for polymorphism, NOT `asChild`
- shadcn/ui v4: When rendering Button as a Link, add `nativeButton={false}` — **every** `render={<Link>}` needs this
- Split components (`equal-split`, `exact-split`, `percentage-split`, `shares-split`): `useEffect` deps must only include user-controlled state (`selected`, `amounts`, etc.) and `totalCents` — never `members` or `onChange` (causes infinite re-renders)
- Dark mode: class-based via `next-themes` ThemeProvider; toggle in sidebar and mobile menu
- Theme: emerald/teal accent color (OKLCH), neutral backgrounds — defined in `globals.css`
- `scripts/dev.mjs` — All-in-one dev script: starts embedded-postgres + Next.js dev server
- `next.config.ts` sets `output: "standalone"` conditionally when `DOCKER_BUILD=1` (set by `docker/Dockerfile`)

## Responsive Layout Architecture

- **Sidebar**: hidden below `lg` (1024px), visible at `lg+` with `lg:sticky lg:top-0 lg:h-dvh`; `overflow-hidden` + `overflow-y-auto` on nav + `shrink-0` on bottom section
- **Outer container**: `min-h-dvh lg:flex lg:h-dvh lg:flex-row` — block flow on mobile (natural scroll), flex on desktop (contained scroll)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sw-carlos-cristobal/sharetab](https://github.com/sw-carlos-cristobal/sharetab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
