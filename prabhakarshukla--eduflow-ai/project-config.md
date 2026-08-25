---
trigger: always_on
description: AI-powered student productivity assistant. Next.js 16 App Router + Supabase (PostgreSQL + Auth + RLS) + Tailwind CSS + Gemini/OpenRouter AI.
---

# EduFlow-AI

AI-powered student productivity assistant. Next.js 16 App Router + Supabase (PostgreSQL + Auth + RLS) + Tailwind CSS + Gemini/OpenRouter AI.

## Skills

Installed in `.agents/skills/`.

| Skill                              | What it does                                                                                    |
| ---------------------------------- | ----------------------------------------------------------------------------------------------- |
| `conventional-commit`              | Reads your git diff and writes a properly formatted Conventional Commit message for you         |
| `core-web-vitals`                  | Audits and fixes LCP, INP, and CLS scores to improve page speed and Google ranking              |
| `deploy-to-vercel`                 | Deploys the app to Vercel (preview by default, production on request)                           |
| `seo-audit`                        | Audits the site for SEO issues — meta tags, crawlability, indexing, page speed, rankings        |
| `supabase-postgres-best-practices` | Reviews and optimizes SQL queries and schema design against Supabase/Postgres performance rules |
| `wcag-audit-patterns`              | Audits the app for WCAG 2.2 accessibility violations and guides fixes                           |

## Commands

```bash
npm run dev       # dev server at localhost:3000
npm run build     # production build (also type-checks, TypeScript strict)
npm run lint      # ESLint via Next.js defaults (no custom config)
```

No test framework is installed. No `test` script exists. No CI/CD workflows.

## Commits

Conventional Commits enforced by commitlint + Husky. Format: `type(scope): description`

- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`, `ci`, `revert`
- Scope optional, lowercase kebab-case. Subject lowercase, imperative, no period, <100 chars.
- Branch naming: `type/short-description` (e.g. `feat/heatmap-calendar`)

## Gotchas

- **`proxy.ts` at root is NOT wired up as middleware.** Next.js only loads `middleware.ts`. This file exports `proxy()` + `config` but is never called by Next.js. If auth guard behavior is needed, rename to `middleware.ts` and export `middleware` instead of `proxy`.
- **Env vars split across two files.** `.env` holds Supabase vars (`NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`). `.env.local` holds AI/email vars (`AI_PROVIDER`, `GEMINI_API_KEY`, `RESEND_API_KEY`). The app checks both at startup via `lib/env-check.ts`.
- **AI graceful degradation.** AI features are disabled in UI when no API keys are configured. Primary provider is Gemini (`AI_PROVIDER=gemini`); auto-falls back to OpenRouter if `OPENAI_API_KEY` is set with `sk-or-` prefix.
- **`.next/` may be committed.** It's in `.gitignore` but the directory exists in the repo. Don't rely on it being fresh; always run `npm run build`.

## Structure

- `app/` — Next.js App Router pages and API routes
- `app/dashboard/` — Authenticated dashboard (sidebar layout, auth guard in layout.tsx)
- `app/api/` — Route handlers (doubt-solver, notes-generator, mood-suggestions, recommendations, contact)
- `components/` — Shared React components
- `context/ThemeContext.tsx` — Dark/light mode (Tailwind `class` strategy)
- `hooks/` — Custom React hooks (realtime chat, theme, feature status, recommendations)
- `lib/ai/agents/` — Multi-agent AI system (7 agents routed by `agent-router.ts`)
- `lib/supabase.ts` — Browser client. `lib/supabase-server.ts` — Server client (cookie-based)
- `supabase/migrations/` — SQL migrations (3 files)
- `@/*` path alias maps to project root

## Style

- Tailwind CSS with extensive CSS custom properties in `globals.css` for theming
- Mobile-first responsive design
- No Prettier or ESLint config files — uses framework defaults
- TypeScript strict mode, `noEmit: true`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [prabhakarshukla/EduFlow-AI](https://github.com/prabhakarshukla/EduFlow-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
