---
trigger: always_on
description: This file is intended for AI coding assistants (Claude Code, GitHub
---

# AGENTS.md - Instructions for AI coding assistants

This file is intended for AI coding assistants (Claude Code, GitHub
Copilot, Cursor, Gemini CLI, ChatGPT, etc.) that a user drops this
repository into. Read this first; it will save you and the user a lot
of back-and-forth.

## What KitPay SolutionIA is

An open source, MIT-licensed, SMS-driven payment infrastructure for
Mauritanian merchants. The user is very likely trying to either:

1. **Deploy their own instance** on their own domain, on their own
   Supabase, to accept payments on their own operator accounts.
2. **Integrate an existing KitPay instance** into their own SaaS or
   e-commerce site.

Read [README.md](./README.md) for the product context in one page.

## Ground rules for AI assistants

- **Never invent an operator, an SMS format, an API endpoint or a table
  name.** If unsure, read the source. The truth lives in `src/app/api/`
  for endpoints, `supabase/*.sql` for the database and
  `src/app/api/sms-ingest/route.ts` for SMS parsers.
- **Never hardcode secrets.** Every secret goes through an env var listed
  in `.env.example`.
- **Never remove the copyright headers or the LICENSE file.**
- **Never suggest replacing Supabase or Next.js with something else** if
  the user has not explicitly asked for a rewrite. The choice is
  deliberate.
- **Never suggest adding a heavy dependency** without justification. The
  runtime dependencies are exactly six: `@supabase/ssr`,
  `@supabase/supabase-js`, `next`, `react`, `react-dom`, `server-only`.
- **Do not use em-dashes, en-dashes, curly quotes, arrows or ellipsis
  Unicode characters in files.** Stick to ASCII plus French accents.

## Repository map for AI navigation

```
kitpay-solutionia/
|-- README.md               Bilingual EN/FR product description
|-- INSTALL.md              Step-by-step deployment guide
|-- AGENTS.md               This file
|-- .env.example            All required env vars
|-- LICENSE                 MIT + trademark notice
|-- SECURITY.md             Vulnerability reporting policy
|-- CONTRIBUTING.md         PR process, code of conduct
|-- SANITIZE.md             Log of what was sanitized before publication
|
|-- src/
|   |-- app/
|   |   |-- api/
|   |   |   |-- v1/                  Public REST API (Bearer kp_* auth)
|   |   |   |-- sms-ingest/          POST from Android SMS forwarder
|   |   |   |-- cron/                Scheduler-triggered endpoints
|   |   |   |   |-- deliver-webhooks/  Webhook worker (this repo's cron)
|   |   |   |   |-- expire-intents/    Marks pending intents expired
|   |   |   |   |-- finalize-pending/  Retries email/telegram for paid intents
|   |   |   |-- admin/               Super-admin only (ADMIN_PASSWORD)
|   |   |   |-- auth/                Supabase auth callbacks
|   |   |   |-- checkout/            POST from home-page catalog
|   |   |   |-- demo/                Public sandbox
|   |   |   |-- intent/              Public read of one intent
|   |   |-- dashboard/               Merchant dashboard (Supabase session)
|   |   |-- admin/                   Super-admin UI
|   |   |-- pay/[ref]/               Hosted checkout page
|   |   |-- payment/[ref]/           Alias, older POC path
|   |   |-- success/[ref]/           Post-payment success page
|   |   |-- onboarding/merchant/     4-step onboarding flow
|   |   |-- signup/, login/          Auth pages
|   |   |-- sandbox/                 Public sandbox
|   |-- lib/
|   |   |-- supabase/                Server/client/middleware helpers
|   |   |-- api/                     REST API primitives (auth, rate limit, etc.)
|   |   |-- hmac.ts                  computeSignature / verifySignature
|   |   |-- email.ts                 sendReceiptEmail via Resend
|   |   |-- telegram.ts              notifyPaymentConfirmed
|   |   |-- types.ts                 Domain types
|   |-- components/                  Shared React components
|
|-- supabase/                Ordered SQL migrations (schema, v2..v36)
|-- docs/                    architecture, api, integration, operators,
|                            android-setup
|-- android/SETUP.md         Same as docs/android-setup.md (legacy path)
|-- public/payments/         Operator logos
|-- tests/                   Vitest + Playwright fixtures
```

## Common tasks a user will ask you to do

### 1. "Deploy this on my Supabase and my Netlify"

Follow [INSTALL.md](./INSTALL.md) end to end. Do not skip step 4 (SQL
migrations) or step 9 (cron scheduler). The webhook worker will not run
without an external cron; the merchant integration will still work
through the `success_url` redirect but no HTTP webhook will be delivered.

### 2. "Integrate KitPay into my Next.js/Django/Laravel/... app"

Point the user to [docs/integration.md](./docs/integration.md). The two
patterns are:

- **Redirect callback** (simpler): create the intent with a
  `success_url` and `cancel_url`. KitPay redirects the customer back to
  you after payment. Verify on your side by calling
  `GET /v1/intents/{ref}` with your Bearer key.
- **Signed HTTP webhook** (async): register a webhook URL in
  `/dashboard/webhooks`. KitPay POSTs signed `payment.succeeded`,
  `payment.expired` and `payment.cancelled` events to your URL. Verify
  with the HMAC snippet from `docs/integration.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MoulayeHamoni/kitpay-solutionia](https://github.com/MoulayeHamoni/kitpay-solutionia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
