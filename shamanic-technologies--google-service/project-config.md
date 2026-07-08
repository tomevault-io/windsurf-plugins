---
trigger: always_on
description: Google Ads API v23 wrapper for MCC agency management, plus Google CRM bronze→silver ingestion (Gmail + People readonly) feeding the dashboard CRM at `/orgs/{orgId}/services/crm`.
---

# google-service

Google Ads API v23 wrapper for MCC agency management, plus Google CRM bronze→silver ingestion (Gmail + People readonly) feeding the dashboard CRM at `/orgs/{orgId}/services/crm`.

## Identity

All endpoints require `x-org-id` and `x-user-id` headers (UUIDs from client-service).
These are the internal org/user identifiers — never use Clerk IDs (clerkOrgId/clerkUserId).
The client-service is the source of truth for identity resolution.

### Tracking / cost-attribution headers

Inbound `x-run-id` (required), `x-feature-slug`, `x-brand-id`, **`x-audience-id`** (all optional) are the tracking block. They are read in `requireIdentityHeaders` (`src/middleware/validate.ts`) onto `req`, then forwarded to every **internal** sibling call (runs-service, billing-service, key-service) via the `trackingHeaders()` allowlist builder in `src/lib/tracking-headers.ts` — never cherry-picked per field. `x-audience-id` (the campaign's priority audience) tags `runs.audience_id` on `createRun` and the cost row on `addCosts`, which is how per-audience cost attribution works (`COALESCE(runs_costs.audience_id, runs.audience_id)` in runs-service). The only metered cost on the campaign path is `serper-dev-query` (`/search/*`). **Egress safety**: `trackingHeaders()` is imported ONLY by the internal clients; external vendor calls (Serper, Gmail/People, Google Ads, Google OAuth) build their own provider-auth headers and MUST never receive the tracking block.

## Stack

See global CLAUDE.md for shared stack details (TypeScript strict, Zod, Vitest+Supertest, Railway).

**Package manager: npm.** Lockfile is `package-lock.json`; the Dockerfile runs `npm ci`. Use `npm install` / `npm test` / `npm run build` locally. Do NOT run `pnpm install` here — it creates a stray `pnpm-lock.yaml` that diverges from the lockfile Railway actually reads.

## OAuth client credentials

The Google OAuth client (Client ID + Secret) is the **same** for the Google Ads Developer Console and the Gmail/People consent flow — one Google Cloud project, one OAuth client. It is registered as platform keys `google-client-id` / `google-client-secret` by the dashboard (`apps/dashboard/src/instrumentation.ts`), not by this service.

Business logic must call `getGoogleOAuthClient()` in `src/services/key-service.ts` to fetch the OAuth client at runtime; never read `GOOGLE_*` env vars directly. If `getGoogleOAuthClient()` returns 404, the dashboard side has not yet registered the providers — fix it there, not here.

## Migrations

`src/db/migrate.ts` exports `runMigrations()` which is awaited from `src/index.ts` **before** `app.listen()`. Every Railway deploy runs the migration; missing tables block startup so a bad migration triggers Railway's restart loop loudly instead of serving 500s.

Schema changes: edit the inline `migration` SQL in `src/db/migrate.ts`. Use `CREATE TABLE IF NOT EXISTS` / `DO $$ ... IF NOT EXISTS ... END $$` so the same migration runs cleanly on every boot.

Manual one-off run still available via `pnpm migrate`, which runs `src/db/migrate-cli.ts`. The CLI runner lives in a **separate file** from `migrate.ts` because `esbuild --bundle --format=cjs` inlines every imported file into `dist/index.js`, and at runtime `require.main === module` evaluates **true** for the bundled entry — so a CLI guard inside `migrate.ts` would fire at boot and call `pool.end()` after migrations, crashing every subsequent request with `Cannot use a pool after calling end on the pool`. Reference: hotfix v0.19.1.

## Data layering

This service owns **bronze** and **silver** for Google CRM data. Gold is served as an additive read projection over bronze+silver (no separate gold table yet).

**Silver tables** (`google_contacts_silver`, `gmail_messages_silver`) are typed projections of the bronze `*_raw` payloads, keyed on the SAME natural key as their bronze source (`(org_id, resource_name)` / `(org_id, gmail_message_id)`). They are:
- **Populated at ingest** — `people-ingest`/`gmail-ingest` parse the payload via `src/services/silver.ts` (`parseContactSilver` / `parseMessageSilver`) and upsert silver right after the bronze upsert (only on inserted/updated bronze rows; deletes cascade to silver via `deleteContactSilver`).
- **Backfilled from bronze on boot** — `src/services/backfill-silver.ts` runs AFTER `app.listen()` (never in the boot window) with `.catch(console.error)`, keyset-paginated + idempotent (upsert), so re-runs and silver schema changes are safe without re-fetching Google.

Bronze stays the source of truth; silver is a rebuildable view. Contact silver columns: `display_name, primary_email, emails[], phones[], organization, job_title, photo_url, updated_at, deleted`. Message silver columns: `from_email, from_name, to_emails[], subject, snippet, sent_at, labels[], history_id`.

### Read endpoints are ADDITIVE (gold)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shamanic-technologies/google-service](https://github.com/shamanic-technologies/google-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
