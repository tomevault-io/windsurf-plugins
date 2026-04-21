---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev        # Start dev server (http://localhost:3000)
npm run build      # Production build
npm run lint       # ESLint
npm test           # Vitest watch mode
npm run test:run   # Vitest single run (CI)
npx tsx ask-pulse-evals/run-evals.ts   # Ask Pulse eval suite (needs OPENAI_API_KEY + SUPABASE_SERVICE_ROLE_KEY)
```

Tests live in `__tests__/` mirroring the source tree. Mock `next/navigation`'s `redirect` to throw so execution stops as in real Next.js: `vi.fn().mockImplementation((url) => { throw new Error(\`NEXT_REDIRECT:\${url}\`) })`.

## Environment Variables

```
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY       # Required for Supabase Storage uploads (JD files, resumes)
GOOGLE_SERVICE_ACCOUNT_EMAIL
GOOGLE_PRIVATE_KEY              # Multiline PEM key
GOOGLE_SHEET_ID
JOOBLE_API_KEY                  # Required for Job Outreach scraper
OPENAI_API_KEY                  # Required for ask-pulse-evals only (evaluation suite)
ANTHROPIC_API_KEY               # Required for Ask Pulse (claude-sonnet-4-6)
GOOGLE_ALUMNI_SHEET_ID          # Alumni roster Google Sheet ID
MCP_DATABASE_URL                # Read-only Postgres URL for the MCP server (pulse_mcp_ro role)
                                # Format: postgresql://pulse_mcp_ro:[PASSWORD]@db.[PROJECT_REF].supabase.co:5432/postgres
                                # See migrations/018_mcp_readonly_role.sql for setup instructions
```

`SUPABASE_SERVICE_ROLE_KEY` must be added manually to Vercel — it is not a `NEXT_PUBLIC_` var and won't be auto-detected.

`MCP_DATABASE_URL` is only needed locally and in any environment running the MCP server. It does not go to Vercel (the MCP server is not deployed there).

## Architecture

### Route Groups

Two top-level route groups with separate layouts and auth contexts:

- **`app/(protected)/`** — Admin and LF users: dashboard, learners, users, placements. Layout at `app/(protected)/layout.tsx` checks `getAppUser()` and redirects learners to `/learner`.
- **`app/(learner)/learner/`** — Learner-facing placement surface: single Dashboard (role feed + filter pills + PlacementSnapshot), role detail, profile/resume. Layout at `app/(learner)/learner/layout.tsx`.

Middleware (`middleware.ts`) enforces authentication on all protected paths and redirects authenticated users away from `/login`. The `protectedPrefixes` array AND the `matcher` config must both be updated when adding new protected routes.

### Identity & Auth

Dual-layer identity:
1. **Supabase Auth** — Google OAuth session; stores email in auth metadata.
2. **`users` table** — App roles (`admin | LF | learner`) and display name.

`getAppUser()` in `lib/auth.ts` (React `cache`-wrapped) looks up the authenticated user's email in the `users` table to get their app role. Every server component and action calls this.

Role enforcement is done in layouts (redirect) and server actions (`requireAdmin()` helper).

In API route handlers, use `createServerSupabaseClient()` directly for auth — do NOT use `getAppUser()`, as React `cache` does not work in route handlers.

### Server Actions Pattern

All mutations use Next.js Server Actions (`'use server'`). Pattern:

```ts
export async function doSomething(formData: FormData) {
  await requireAdmin()                          // role check + redirect
  const supabase = await createServerSupabaseClient()
  await supabase.from('table').insert(...)
  revalidatePath('/affected/path')              // clear Next.js cache
}
```

All pages that read mutable data use `export const dynamic = 'force-dynamic'`.

### Supabase Clients

- `lib/supabase.ts` — browser client (for client components)
- `lib/supabase-server.ts` — server client with SSR cookie handling (for server components and actions)
- For storage operations requiring elevated privileges, create an admin client directly with `createClient(url, SUPABASE_SERVICE_ROLE_KEY)` — see `uploadJdAttachment()` in `app/(protected)/placements/actions.ts`.

### Placements Module

Admin-facing (`app/(protected)/placements/`) and learner-facing (`app/(learner)/learner/`) share the same Supabase tables:

- `companies` → `roles` → `applications` (cascade delete)
- `resumes` — stored in `resumes` Supabase Storage bucket at `{user_id}/{timestamp}.pdf`
- `jd-files` — stored in `jd-files` Supabase Storage bucket at `{role_id}.pdf`
- `role_preferences` — learner "not interested" signals with reasons

Application status progression: `applied → shortlisted → hired` with two dropout statuses:
- `not_shortlisted` — company did not select for interview (pre-shortlist dropout)
- `rejected` — rejected after interview (post-shortlist dropout)

`applications` also stores `not_shortlisted_reason` and `rejection_feedback` (nullable TEXT). Admin sets these via a required modal when changing to those statuses; they are displayed to the learner on the role detail page.

Company display order is managed via `sort_order` column, with drag-to-reorder in `CompaniesListClient.tsx` (dnd-kit).

### Navigation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gayathri-meka) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
