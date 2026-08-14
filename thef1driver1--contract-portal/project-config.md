---
trigger: always_on
description: A bilingual (ES/EN) rental contract management SaaS for Puerto Rico landlords. Landlords generate, send, and track lease agreements; tenants receive email invites to sign digitally. Includes investment analytics, expense tracking, and tax reporting (Schedule E).
---

# Contract Portal — Claude Code Context

## What This App Is

A bilingual (ES/EN) rental contract management SaaS for Puerto Rico landlords. Landlords generate, send, and track lease agreements; tenants receive email invites to sign digitally. Includes investment analytics, expense tracking, and tax reporting (Schedule E).

**Live stack:** Next.js 14 (App Router) · Supabase (Postgres + Auth + Storage) · Resend (email) · Twilio (SMS) · Stripe (billing) · Vercel (hosting) · Upstash Redis (rate limiting)

**Default locale:** Spanish (`es`). Middleware reads `NEXT_LOCALE` cookie; tenant-facing UI defaults to `es`.

---

## Repo Layout

```
Contract_Portal/
├── web/                  ← Next.js app (all active development happens here)
│   ├── app/              ← App Router routes + API routes
│   ├── components/       ← Shared UI components
│   ├── lib/              ← Business logic, Supabase clients, types, schemas
│   ├── messages/         ← i18n JSON (en.json, es.json)
│   ├── templates/        ← DOCX lease template
│   └── supabase/         ← Migration files + schema reference
├── supabase/             ← Root-level migrations (PR expansion)
├── PLANS/                ← Feature roadmap docs
└── CLAUDE.md             ← This file
```

---

## Route Map

### Auth (`web/app/(auth)/`)
| Route | File |
|---|---|
| `/login` | `(auth)/login/page.tsx` |
| `/signup` | `(auth)/signup/page.tsx` |
| `/forgot-password` | `(auth)/forgot-password/page.tsx` |
| `/reset-password` | `(auth)/reset-password/page.tsx` |

### Landlord Dashboard (`web/app/(dashboard)/`)
| Route | Purpose |
|---|---|
| `/dashboard` | Main overview |
| `/contracts` | Contract list |
| `/contracts/new` | Create contract |
| `/contracts/[id]` | Contract detail, actions, notifications |
| `/properties` | Property CRUD |
| `/tenants` | Tenant CRUD |
| `/expenses` | Expense tracking (tax deductions) |
| `/market` | Market analysis (Zillow data) |
| `/market/[id]` | Property market detail |
| `/watchlist` | Investment watchlist |
| `/watchlist/[id]/analyze` | Investment ROI calculator |
| `/reports/schedule-e` | IRS Schedule E PDF generation |
| `/settings/billing` | Stripe plan management |
| `/settings/managers` | Property manager invitations |
| `/settings/notifications` | SMS/email reminder rules |
| `/settings/sections` | Custom contract clause library |
| `/settings/templates` | DOCX template uploads |
| `/profile` | User profile |

### Tenant Portal (`web/app/(portal)/`)
| Route | Purpose |
|---|---|
| `/portal` | Tenant dashboard |
| `/portal/sign/[contractId]` | Digital signature flow |

### Public
| Route | Purpose |
|---|---|
| `/invite/[token]` | Tenant invite + signup |
| `/invite/manager/[token]` | Property manager invite |
| `/` | Landing page |
| `/pricing` | Pricing page |

### API (`web/app/api/`)
Key endpoints — all authenticated unless noted:
- `POST /api/contracts` — create contract
- `GET/PUT/DELETE /api/contracts/[id]`
- `POST /api/contracts/[id]/send-email`
- `POST /api/contracts/[id]/quick-sms`
- `POST /api/contracts/[id]/invite` — email invite to tenant
- `POST /api/generate` — DOCX/PDF generation (uses `docxtemplater`)
- `GET/POST /api/properties`
- `GET/POST /api/tenants`
- `GET/POST /api/expenses`
- `GET/POST /api/templates` — custom DOCX templates
- `GET/POST /api/managers` + `POST /api/managers/invite/[token]`
- `GET /api/market/properties` — Zillow market data
- `POST /api/investment/[watchlistId]` — ROI calculation
- `GET /api/reports/schedule-e` — Schedule E PDF
- `POST /api/billing/checkout` — Stripe checkout
- `POST /api/billing/portal` — Stripe customer portal
- `POST /api/webhooks/stripe` — Stripe webhook (no auth, HMAC verified)
- `POST /api/cron/notify` — Scheduled expiry reminders (cron, Bearer auth)
- `GET/POST /api/invite/[token]` — public, no auth
- `POST /api/portal/contracts/[id]/sign` — tenant signature submission

---

## Key Source Files

| File | What It Does |
|---|---|
| `web/lib/types.ts` | All TypeScript types and interfaces. Source of truth for domain models. |
| `web/lib/schemas.ts` | Zod validation schemas for all API request bodies |
| `web/lib/supabase-server.ts` | `createClient()` (RLS-aware SSR) and `createAdminClient()` (service role, bypasses RLS) |
| `web/lib/supabase.ts` | Client-side Supabase client |
| `web/lib/contract-context.ts` | `buildContext(contract)` — maps DB contract → DOCX template variables (Spanish) |
| `web/lib/notify.ts` | `sendResendEmail()`, `sendTwilioSms()`, `sendTenantInviteEmail()` |
| `web/lib/pdf-react.tsx` | React PDF renderer for contracts |
| `web/lib/pdf-template.ts` | DOCX → PDF pipeline |
| `web/lib/generate-docx.ts` | DOCX generation from `docxtemplater` |
| `web/lib/investment.ts` | Cap rate, cash flow, ROI calculation helpers |
| `web/lib/subscription.ts` | Plan limit checks (`PLAN_LIMITS`) |
| `web/lib/rate-limit.ts` | Upstash Redis rate limiter wrapper |
| `web/middleware.ts` | Auth routing, role-based redirects, locale cookie |
| `web/components/ContractBuilder.tsx` | Large form for contract create/edit (61KB) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheF1Driver1/Contract_Portal](https://github.com/TheF1Driver1/Contract_Portal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
