---
trigger: always_on
description: Next.js App Router dashboard for Kith Climate CRM. Deployed on Cloudflare Workers with webhook processing via Supabase Edge Functions. Supabase backend on project `tfcuozmbnnswencikncv` ("AI Context"), schema `kith_climate`. Auth via Auth0. LinkedIn enrichment via Apify.
---

# Kith Climate CRM Dashboard

## Overview

Next.js App Router dashboard for Kith Climate CRM. Deployed on Cloudflare Workers with webhook processing via Supabase Edge Functions. Supabase backend on project `tfcuozmbnnswencikncv` ("AI Context"), schema `kith_climate`. Auth via Auth0. LinkedIn enrichment via Apify.

## Tech Stack

- **Next.js** (App Router) on **Cloudflare Workers**
- **Supabase** (PostgreSQL, Edge Functions, RLS)
- **Auth0** for authentication
- **Stripe** for payments
- **Fathom.ai** for interview recording
- **Calendly** for booking
- **Apify** for LinkedIn enrichment
- **Resend** for email delivery

## Architecture

The system has two runtime environments:

### Next.js Dashboard (Cloudflare Workers)
- Serves the CRM dashboard UI
- API routes: `/api/fathom/backfill`, `/api/outlook/sync`, `/api/leads`, `/api/enrichment/status`, `/api/emails`
- Accesses Fathom API keys via `process.env` (configured in Cloudflare)

### Supabase Edge Functions (Deno)
- `fathom-webhook` — Fathom `new_meeting_content_ready` events → classify, upsert interviews, advance funnel (cohort-aware)
- `fathom-backfill` — Manual backfill: fetches all Fathom meetings, matches to existing interviews or creates new (cohort-aware)
- `calendly-webhook` — Calendly `invitee.created`/`invitee.canceled` → manage bookings and funnel (cohort-aware)
- `backfill-calendly` — Manual backfill: fetches Calendly events, deduplicates, inserts bookings (cohort-aware)
- `outlook-daily-sync` — Daily Outlook email scan → detects interview/enrolment invites → creates pending funnel changes (cohort-aware)
- `stripe-kith-climate-webhook` — Stripe `checkout.session.completed`/`charge.refunded` → manage payments (cohort-aware)
- `kith-climate-email-automation` — DB trigger for funnel changes → queue/send automated emails (receives cohort from trigger)
- `kith-climate-send-email` — Send emails via Resend, personalise with `{cohort}` variable, record in emails table
- `kith-climate-pending-email-review` — Approve/reject pending emails (called from dashboard via proxy)

### Architecture Principles
- **Database mutations on RLS-protected tables must go through Supabase Edge Functions**, not Cloudflare Workers API routes. Dashboard uses anon key (subject to RLS); edge functions use service role key (bypasses RLS).
- **Next.js API routes should be thin proxies** for operations requiring service role key. Pattern: fetch to `${SUPABASE_URL}/functions/v1/<function-name>` with anon key auth. See `trigger-sync/route.ts` for reference.
- **Never add SUPABASE_SERVICE_ROLE_KEY to Cloudflare Workers.** Create or extend a Supabase edge function instead.

## Multi-Cohort System

The funnel is **cohort-aware**. Each customer can exist in multiple cohorts simultaneously (e.g., deferred from March, re-applying for May).

### Two status fields on `customers`

- **`cohort_statuses`** (JSONB) — source of truth per cohort: `{"March 16th 2026": {"status": "deferred_next_cohort", "updated_at": "..."}, "May 18th 2026": {"status": "invited_to_enrol", "updated_at": "..."}}`
- **`funnel_status`** (text) — auto-calculated as the highest-ranked status across all cohort entries. Used only for the "All Cohorts" view.

### Cohort assignment rule

**Every function that assigns a cohort must derive it from the customer's `cohort_statuses`**, picking the entry with the most recent `updated_at`. A hardcoded `DEFAULT_COHORT` is only used as a last resort for brand-new customers with no cohort history.

All edge functions, API routes, and frontend components follow this pattern. **Never hardcode a cohort string** except as a fallback constant.

### The `advance_funnel` RPC

All funnel transitions go through the `kith_climate.advance_funnel(p_customer_id, p_new_status, p_cohort)` database RPC:
- When `p_cohort` is provided: updates the specific cohort entry in `cohort_statuses`, then recalculates `funnel_status` as the highest rank across all entries.
- When `p_cohort` is NULL: legacy path — updates only `funnel_status`. **Avoid this.**

### Adding a new cohort

When a new cohort starts, update these locations:

| Location | What to change |
|----------|---------------|
| `src/lib/supabase.ts` | Add entry to `COHORT_OPTIONS` array (all components import from here) |
| `src/components/CohortSelector.tsx` | Update default `??` fallback |
| `src/app/funnel/page.tsx` | Update default `??` fallback |
| `stripe-kith-climate-webhook/index.ts` | Update `CURRENT_COHORT` fallback + redeploy |
| `outlook-daily-sync/index.ts` | Update `DEFAULT_COHORT` fallback + redeploy |
| `calendly-webhook/index.ts` | Update `DEFAULT_COHORT` fallback + redeploy |
| `fathom-webhook/index.ts` | Update `DEFAULT_COHORT` fallback + redeploy |
| `backfill-calendly/index.ts` | Update `DEFAULT_COHORT` fallback + redeploy |
| `fathom-backfill/index.ts` | Update `DEFAULT_COHORT` fallback + redeploy |

### Cohort-filtered views

The CohortSelector (URL param `?cohort=May 18th 2026`) controls which cohort the dashboard shows. When a cohort is selected:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/voiz-academy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
