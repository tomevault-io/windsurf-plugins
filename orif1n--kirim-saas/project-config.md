---
trigger: always_on
description: Modular, type-safe SaaS starter. Bun monorepo, Hono API, Vite/React web, Drizzle + Postgres, Better Auth, provider-agnostic payments, hardened by default (OWASP audit passing).
---

# saas-boilerplate

Modular, type-safe SaaS starter. Bun monorepo, Hono API, Vite/React web, Drizzle + Postgres, Better Auth, provider-agnostic payments, hardened by default (OWASP audit passing).

Read this file first. Then read the AGENTS.md next to whatever you're about to edit — each package documents its local invariants.

## Repository map

```
saas-boilerplate/
├── apps/
│   ├── api/          Hono + zod-openapi + Better Auth handler (Bun runtime)
│   ├── web/          Vite + React 19 + shadcn/ui + TanStack Router + i18next
│   └── worker/       Bun + BullMQ background jobs (billing reconciler, subscription lifecycle)
├── packages/
│   ├── config/       tsconfig presets + Zod env schema
│   ├── shared/       Enums, AppError, Result, crypto helpers — zero I/O, safe everywhere
│   ├── db/           Drizzle schema, client, migrations, seed
│   ├── auth/         Better Auth server factory + browser client
│   ├── email/        Resend mailer + React Email templates
│   ├── payments/     Provider interface + Duitku adapter (Midtrans/Xendit/Stripe are interface-only stubs that throw until implemented)
│   └── storage/      Provider interface + R2/S3 adapter — presigned uploads, magic-byte verify
├── docker-compose.yml    Postgres + Redis + Mailpit for local dev
├── Dockerfile.api        Multi-stage Bun runtime (non-root)
├── Dockerfile.web        Multi-stage nginx-unprivileged (port 8080)
├── renovate.json         Grouped dependency PRs, auto-merge safe minors
└── .github/workflows/    CI (lint/typecheck/test + e2e) and Docker publish to GHCR
```

## Golden rules

1. **No circular deps.** `apps/*` depends on `packages/*`, never the reverse. `packages/db` and `packages/shared` are leaf packages with no cross-package runtime deps.
2. **No I/O at import time.** Every package exports pure factories (`createXxx(config)`). The API's `server.ts` is the ONE place that reads env and constructs services.
3. **Money is integer minor units.** `priceAmount`, `amount` are integers. Currency is a separate column. No floats, ever.
4. **Payments are provider-agnostic.** Routes talk only to the `PaymentProvider` interface. Adding a provider = one file + one factory branch. Swapping providers = one env var.
5. **Errors are `AppError` at package boundaries.** The API error middleware is the ONLY response formatter. Frontend surfaces them via `normalizeError()` + i18n code table.
6. **Type-safe env.** `@saas/config/env` validates once at boot and throws on missing/invalid vars. Placeholder values in `.env.example` are actively rejected.
7. **OpenAPI is source of truth for the HTTP contract.** Every route uses `createRoute(...)`. `/api/openapi.json` and `/api/docs` are gated OFF in production.
8. **i18n covers every user-visible string.** Locales are split per namespace under `apps/web/src/i18n/locales/{en,id}/<ns>.ts`. Add keys to BOTH `en/<ns>.ts` and `id/<ns>.ts` in the same commit. The `id` aggregator is typed against `typeof en`, so a missing key fails typecheck instead of silently falling back. Error codes are looked up via `errors.<CODE>` in `errors.ts`.
9. **Column-level encryption for secrets at rest.** Two independent mechanisms, do NOT conflate them: (a) **OAuth tokens** on `accounts.*` are encrypted by Better Auth's built-in `account.encryptOAuthTokens` (its own ciphertext format, keyed off `AUTH_SECRET`) — NOT by `@saas/shared/crypto`, whose format Better Auth's OAuth refresh path cannot read. (b) **Verification tokens** (`verifications.value`) and any third-party integration secrets go through `@saas/shared/crypto` (AES-256-GCM), gated on `COLUMN_ENCRYPTION_KEY` — when that key is unset the hook is a no-op and values are stored plaintext (dev default; production MUST set it — the env schema throws at boot without it). See `packages/auth/AGENTS.md` "Column-level encryption". Personal access tokens (`api_keys.hashed_key`) use `hashApiKey` from `@saas/shared/crypto`, which returns a raw 32-byte `Buffer` (sha256 digest) mapped to a `bytea` column — NOT hex. Plaintext is returned to the caller ONCE on create and NEVER persisted or logged.
10. **Webhooks: verify signature, re-check business fields, transact.** Never trust that a signed payload is safe to write — cross-check amount/currency against the pending row. Persist raw payloads ONLY through the adapter's `redactWebhookPayload()` — the DB is not a place for provider signatures, customer PII, or free-form fields. Every `PaymentProvider` MUST implement `redactWebhookPayload(raw)`; per-provider allowlists are the invariant. Redacted payloads land in the sibling table `payment_webhook_events` (append-only), NOT on the `payments` row — that keeps the hot `/billing/payments` history query narrow. See `packages/payments/AGENTS.md`.
11. **Tenant scoping is a security invariant, not a filter.** Every read/write on business data MUST include `WHERE organization_id = ?`. The active org is resolved via `requireActiveOrg(services, session)` — never trust the client. Missing the predicate is a cross-tenant data leak, not a bug.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [orif1n/kirim-saas](https://github.com/orif1n/kirim-saas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
