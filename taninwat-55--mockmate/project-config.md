---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Monorepo Structure

This is a pnpm + Turborepo monorepo. All commands should be run from the repo root unless targeting a specific workspace.

```
mockmate/
├── apps/
│   ├── web/        ← Next.js app (all MVP work happens here)
│   └── mobile/     ← Placeholder, not yet scaffolded
├── packages/
│   └── db/         ← Shared Prisma package: schema, client singleton, types
├── turbo.json
└── pnpm-workspace.yaml
```

Node is pinned via `.nvmrc` (24) and `engines`. pnpm is pinned via `packageManager` (Corepack enforces it). There is exactly ONE lockfile, at the root — never commit a nested `pnpm-lock.yaml`.

## Commands

All run from repo root:

```bash
pnpm dev          # start Next.js dev server (apps/web only currently)
pnpm build        # build all workspaces via Turborepo
pnpm lint         # lint all workspaces
```

Database (run from root, delegates to the `@mockmate/db` workspace):

```bash
pnpm db:generate   # regenerate Prisma client after editing the schema
pnpm db:push       # push schema to Neon DB (dev, no migration history)
pnpm db:migrate    # create + apply a migration
pnpm db:studio     # open Prisma Studio GUI
```

These map to `prisma` commands run inside `packages/db`. If you ever need to call the Prisma binary directly, run it from `packages/db` (that's where the schema and `prisma.config.ts` live), and use `./node_modules/.bin/prisma` rather than `pnpm prisma` to avoid the dep-status check.

## Tech Stack

| Layer | Choice | Why |
|---|---|---|
| Framework | Next.js 16 App Router | API routes + UI in one deploy, streaming support |
| Database | Neon PostgreSQL + Prisma v7 | Serverless Postgres, type-safe queries |
| Auth | NextAuth v5 beta + Google OAuth | Google-only for MVP, no password storage |
| AI | Vercel AI SDK + `@ai-sdk/google` | Provider abstraction — swap Gemini for GPT/Claude by changing one import |
| LLM | Google Gemini Flash | Free tier (1,500 req/day), sufficient for MVP |
| Analytics | PostHog | Events: `session_started`, `session_completed`, `feedback_rated` |
| Email | Resend via AWS Lambda | Async post-session summary, non-blocking |
| Styling | Tailwind v4 + PostCSS | PostCSS required by Next.js; not needed in Vite-based projects |

## Key Architecture Rules

**Never call the Gemini SDK directly.** All LLM calls go through Vercel AI SDK (`import { streamText, generateObject } from 'ai'`). Provider-specific imports are only `@ai-sdk/google`. See `docs/decisions/002-vercel-ai-sdk-abstraction.md`.

**Save the user's answer to DB before calling the LLM.** If the LLM call fails or times out, the answer must not be lost. The DB write happens first, always.

**Grading uses `evaluationNote` fields, not the conversation.** After each question, the AI appends a hidden evaluation note to the Question record. The final grading matrix is generated from those 5 notes — not by re-reading the full message history.

**`Session` ≠ `InterviewSession`.** The Prisma schema has both. `Session` is NextAuth's auth cookie table (do not rename). `InterviewSession` is the application's interview record.

## Coding Standards

**TypeScript**
- `strict` is on. No `any` — use a proper type, or `unknown` and narrow.
- Define interfaces/types for component props, API responses, and data shapes. Let inference handle the obvious; add explicit types where they aid clarity.

**React / Next.js (App Router)**
- Server Components by default. Add `"use client"` only for interactivity, hooks, or browser APIs — push it to the leaves of the tree, not whole pages.
- **Server Actions** for form submissions and simple mutations.
- **API Routes** for: the AI streaming endpoints, webhooks (Stripe etc.), long-running operations, responses needing specific status/headers, and any endpoint a future mobile/CLI client will call.
- Functional components only. Extract reusable logic into custom hooks. One job per component.

**File organization** (inside `apps/web/src/`)
- Components: `components/<feature>/ComponentName.tsx`
- Pages: `app/<route>/page.tsx`
- Server Actions: `actions/<feature>.ts`
- Types: `types/<feature>.ts`
- Utils/helpers: `lib/<utility>.ts`

**Naming**
- Components & types/interfaces: `PascalCase` (no `I` prefix). Functions: `camelCase`. Constants: `SCREAMING_SNAKE_CASE`. Files: match the component name, else `kebab-case`.

**Styling**
- Tailwind for all styling. No inline styles.
- **Shadcn/ui** for components (install when the first real UI is built); use its `sonner` for toasts.
- Light mode first, dark mode as an option.

**Validation — Zod**
- Validate all external input with Zod: Server Action inputs, API route bodies, and LLM structured output (`generateObject` schema for the grading matrix).

**Data fetching**
- Server Components read directly via `prisma` from `@mockmate/db`. Client Components mutate via Server Actions — never call Prisma from the client.

**Error handling (Server Actions)**
- Wrap in `try/catch`. Return a `{ success, data?, error? }` shape. Surface failures to the user via a toast, not a thrown error.

**Database workflow (phased — we are in phase 2 as of #29)**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Taninwat-55/mockmate](https://github.com/Taninwat-55/mockmate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
