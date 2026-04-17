---
trigger: always_on
description: KinOS is an internal CRM replacing Enerflo for KIN Home's solar sales operations.
---

# KinOS — Solar CRM for KIN Home

## What This Project Is

KinOS is an internal CRM replacing Enerflo for KIN Home's solar sales operations.
Built with Next.js 16 (App Router), Supabase (Postgres + Auth + RLS), deployed on Vercel.
Epics 0–10 are complete (Auth, RepCard, Pipeline, Leads, Calendar, Aurora Design, Proposal/Pricing, Financing, Document Signing + Notifications, Submission & Gating).

## Before You Start ANY Task

Read these docs before making changes — not just for "architectural decisions", but for ANY non-trivial work.
The goal: every agent understands what exists, what we're building toward, and how current work fits in.

**Understand the vision and future:**

- `/docs/kinos-vision-and-state.md` — What we're building, why, current state, architecture decisions, what's next
- `/docs/kinos-future-features.md` — Detailed explainers for ALL planned features with build constraints. READ BEFORE BUILDING — understand how your work impacts future features.

**Understand what exists today:**

- `/docs/PROJECT-KNOWLEDGE.md` — Master knowledge base: schema, migrations, API routes, epic status. **Single source of truth.**
- `/docs/db-audit.md` — Live database inventory with column summaries (51 tables + 2 views, verified 2026-02-12)
- `/docs/schema-reference.md` — Column-level schema from live DB (51 tables, 952 columns)

**Architecture reference:**

- `/docs/blueprint.md` — Full system architecture spec (reference, may lag behind actual state)

## Tech Stack

- **Framework:** Next.js 16 with App Router (NOT Pages Router). Middleware file is `proxy.ts` not `middleware.ts`.
- **Database:** Supabase (Postgres) with Row Level Security. 51 tables + 2 views live.
- **Auth:** Supabase Auth (email/password). No self-registration. Admin creates accounts.
- **Styling:** Tailwind CSS + shadcn/ui components
- **Language:** TypeScript (strict mode)
- **Package Manager:** pnpm
- **Deployment:** Vercel at kin-os-one.vercel.app (auto-deploys from main branch)

## Database Access Patterns

- **Browser/Client components:** Use `lib/supabase/client.ts` (anon key, respects RLS)
- **Server components/actions:** Use `lib/supabase/server.ts` (cookie-based auth, respects RLS)
- **Webhooks/API routes:** Use `lib/supabase/admin.ts` (service role, bypasses RLS)
- NEVER use the service role client in client components
- NEVER expose the service role key to the browser

## Pipeline Stages (19 stages — blueprint §9)

```
new_lead → appointment_set → appointment_sat → design_requested →
design_complete → proposal_sent → proposal_accepted → financing_applied →
financing_approved → stips_pending → stips_cleared → contract_sent →
contract_signed → submission_ready → submitted → intake_approved
                                                → intake_rejected
Also: cancelled, lost
```

Stage constants are in `lib/constants/pipeline.ts`. Do NOT add, remove, or rename stages without explicit instruction.

## Key Architecture Rules

- All data is company-isolated via RLS policies using `company_id`
- Soft deletes everywhere (`deleted_at` column, never hard delete)
- Major tables have `created_at`, `updated_at`, `updated_by`
- Deal stage transitions auto-logged by Postgres trigger to `deal_stage_history`
- Contact field changes auto-logged by Postgres trigger to `contact_change_history`
- Deal numbers auto-generate as `KIN-{YEAR}-{SEQ}` via Postgres trigger

## Code Conventions

- API routes go in `app/api/` — webhooks, CRUD endpoints
- Use server components by default, `"use client"` only when needed
- Error handling: always try/catch Supabase calls, return typed results
- No `any` types — use generated Supabase types from `lib/supabase/database.types.ts`
- Webhook handlers: always use `supabaseAdmin`, log to `webhook_events`, return 200 quickly

## Integrations Status

- **RepCard:** ✅ Complete — 7 webhook handlers, user sync, contact creation
- **Aurora Solar:** ✅ Complete — API client, service layer, webhooks, 3 design paths
- **Lender APIs:** 📋 Planned — 6 lenders with API access (GoodLeap, LightReach, Sunlight, Dividend, EnFin, Skylight)
- **Arrivy:** 📋 Planned — Site survey scheduling, field management, install visibility
- **Document Signing:** ✅ Manual tracking complete — ManualSigningProvider, PandaDoc adapter interface ready for Phase 2
- **Submission:** ✅ Complete — ManualSubmissionProvider, SubmissionPayload interface, deal_snapshots, gate engine (13 blueprint gates)
- **Quickbase:** 📋 Planned — QuickbaseSubmissionProvider implementation for actual push + post-sale bidirectional sync
- **Sequifi/CaptiveIQ:** 📋 Planned — Commission push on deal milestones
- **Twilio:** 📋 Planned — SMS/email notifications

## Build With the End in Mind

Before building any feature, read `docs/kinos-future-features.md` to understand how your work impacts planned features. Key principles:

- Every external integration uses an adapter pattern (swap providers without touching UI)
- Every deal state change should emit a notification event (even if notification system isn't built yet)
- Every admin-configurable value should have an audit trail (who changed what, when)
- Every new table needs RLS policies from day one
- Mobile-responsive layouts from the start — no desktop-only patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/avelkins10) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
