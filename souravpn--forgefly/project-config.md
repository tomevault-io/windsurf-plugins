---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev          # Start dev server
pnpm build        # Production build
pnpm test         # Run tests (watch mode)
pnpm test:ci      # Run tests with coverage (single run)
pnpm test:watch   # Run tests in watch mode
pnpm test:ui      # Run tests with Vitest UI
pnpm lint         # Full lint: type-check + biome + tailwind CSS check
pnpm type-check   # TypeScript type checking only
pnpm format       # Format with Prettier
```

To run a single test file:
```bash
pnpm vitest run src/test/stripe.test.ts
```

Supabase CLI (project is **Forgefly-prod** — there is no separate local/staging project; always confirm with the user before pushing migrations or deploying functions):
```bash
supabase migration list          # compare local migrations against the remote
supabase db push                 # apply pending migrations to Forgefly-prod
supabase functions deploy <name> # deploy a single edge function
```

## Environment

Requires a `.env` file with:
```
VITE_SUPABASE_URL=...
VITE_SUPABASE_ANON_KEY=...
```

Supabase Edge Functions require secrets set server-side (Supabase dashboard → Edge Functions → Secrets), grouped by what they power:
- **Core**: `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY` / `SUPABASE_SERVICE_KEY`, `SITE_URL`
- **AI**: `ANTHROPIC_API_KEY` (Freeda, promotion captions), `OPENAI_API_KEY` (gpt-image-2 promotion images), `PERPLEXITY_API_KEY` (Sonar search-grounded research, market research pipeline)
- **Payments**: `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `STRIPE_AGENCY_MONTHLY_PRICE_ID`, `STRIPE_AGENCY_YEARLY_PRICE_ID`
- **Social**: `META_APP_ID` / `META_APP_SECRET` (Instagram + Facebook + WhatsApp OAuth and publishing, one Meta app for all three), `INSTAGRAM_APP_ID` / `INSTAGRAM_APP_SECRET`, `WHATSAPP_WEBHOOK_VERIFY_TOKEN`
- **Promotions video**: `SHOTSTACK_API_KEY` (currently pinned to Shotstack's free/watermarked sandbox endpoint — swap to a production key + endpoint before relying on unwatermarked Reels)
- **Misc**: `RESEND_API_KEY` (email), `REVIEW_JWT_SECRET` (review-link tokens), `APPLE_TEAM_ID` / `APPLE_PASS_TYPE_ID` (wallet pass generation)

## Architecture

**Forgefly** is an AI-powered business OS for freelancers/solopreneurs. React 18 + TypeScript SPA with Supabase as the backend, Stripe for payments, and Meta/Shotstack integrations for social promotion.

### Auth & routing

- `AuthContext` (`src/contexts/AuthContext.tsx`) manages Supabase session, user `Profile`, and `Subscription` (freelancer vs. agency tier). Authentication uses a `username@miaoda.com` email convention — users sign in with a username, not an email.
- `RouteGuard` (`src/components/common/RouteGuard.tsx`) redirects unauthenticated users to `/login`. Public routes are declared in `routes.tsx` via `public: true`.
- `App.tsx` splits routes into two groups: public routes render without a layout wrapper; protected routes render inside `MainLayout` (Sidebar + Outlet + AICopilot).
- Notable public routes beyond auth: `/preview` (pre-signup AI-generated business preview, `GeneratedPortalPage.tsx` — a deliberately standalone read-only clone, not a reuse of the real dashboard; see `src/components/preview/` if touching this), `/p/:slug` (a freelancer's public portfolio), `/portal/:token` (client portal), `/documentation` (in-app help docs), `/contact`.

### Data layer

All DB access goes through `src/services/` — one file per domain: `clientService.ts`, `projectService.ts`, `invoiceService.ts`, `proposalService.ts`, `paymentService.ts`, `packageService.ts`, `calendarService.ts`, `financeService.ts`, `timeService.ts`, `portalFileService.ts`, `dashboardService.ts` (cross-cutting reads for the Dashboard and Freeda's KPI catalog — single source of truth so numbers can't drift between the two), `socialService.ts` (platform connections, competitor intel), `promotionService.ts` (AI-generated promotion drafts and publishing). Services call Supabase directly via `src/db/supabase.ts`. Several services expose a `subscribeTo*` function using Supabase Realtime for live updates.

All domain types are in `src/types/types.ts` — `Client`, `Project`, `Invoice`, `Proposal`, `Payment`, `Package`, `CalendarEvent`, `Automation`, `Profile`, `BusinessProfile`, plus social/promotion types (`SocialConnectionStatus`, `Promotion`, `SocialPostTargetPlatform`).

### Supabase Edge Functions

Located in `supabase/functions/`, written in Deno/TypeScript. Grouped by area (see each function's `index.ts` for detail):

| Area | Functions |
|---|---|
| **Freeda / AI** | `ai-gateway` (sole LLM entry point — see Freeda section below), `extract-receipt`, `generate-visibility-kit`, `research-company`, `research-competitor`, `quarterly-review-insight`, `trigger-nudges`, `generate-market-research` (Perplexity Sonar research + Claude synthesis, see below) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [souravpn/forgefly](https://github.com/souravpn/forgefly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
