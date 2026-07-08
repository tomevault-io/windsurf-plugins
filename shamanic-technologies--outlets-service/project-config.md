---
trigger: always_on
description: | Tier | Prefix | Auth | Example |
---

# outlets-service-v1

## Route Structure

| Tier | Prefix | Auth | Example |
|------|--------|------|---------|
| Public | `/` | None | `GET /health`, `GET /openapi.json` |
| Internal | `/internal/*` | `x-api-key` | `POST /internal/enrich` |
| Org-scoped | `/orgs/*` | `x-api-key` + `x-org-id` | `GET /orgs/outlets`, `POST /orgs/outlets/discover` |

## Auth Model

Two middlewares, applied in `app.ts`:
1. **`apiKeyAuth`** — validates `x-api-key` against `OUTLETS_SERVICE_API_KEY`. Crashes at startup if env var missing.
2. **`requireOrgId`** — requires `x-org-id`, parses all other identity headers as optional into `req.orgContext` (`OrgContext` type).

Middleware order in `app.ts`:
```
Public routes (no middleware)
  └─ apiKeyAuth
       ├─ /internal/* (API key only)
       └─ requireOrgId
            └─ /orgs/* (API key + org context)
```

## Tenant Isolation

Every `/orgs/*` SQL query MUST include `WHERE org_id = $N` — no exceptions. This prevents cross-org data leaks.

## OrgContext Type

```typescript
interface OrgContext {
  orgId: string;        // required (from x-org-id)
  userId?: string;      // optional (from x-user-id)
  runId?: string;       // optional (from x-run-id)
  campaignId?: string;  // optional (from x-campaign-id)
  brandIds: string[];   // optional (from x-brand-id, comma-split)
  featureSlug?: string; // optional (from x-feature-slug)
  workflowSlug?: string;// optional (from x-workflow-slug)
}
```

## Header Forwarding

`buildServiceHeaders(apiKey, ctx)` in `src/services/headers.ts` — includes `x-api-key` and `x-org-id` unconditionally, all others only when present.

## Cross-service enrichment: fail-loud vs best-effort

Two classes of cross-service read enrichment, and they have OPPOSITE failure modes — don't apply one rule to both:

- **Core data → fail-loud.** Enrichment the endpoint's purpose depends on (e.g. journalist outreach status from journalists-service on `GET /orgs/outlets`, which drives `byOutreachStatus`). On failure, throw → 500. `fetchOutletStatuses` is the reference.
- **Decorative annotation → best-effort.** Enrichment that's nice-to-have (e.g. Domain Rating from ahref-service). On failure, `console.warn` + serve the field as `null`. Never 500 the endpoint for it. `getDrStatus` in the route handlers is the reference (the ahref *client* stays fail-loud; the *route* catches and degrades).

**Before choosing fail-loud for a cross-service read, confirm the dependency is deployed in EVERY target environment.** A prod-only dependency + fail-loud read = the non-prod environment 500s the whole endpoint. `ahref-service` is **prod-only** (no staging); Railway private DNS is environment-scoped, so `ahref-service.railway.internal` does not resolve from the staging environment. That's exactly why the DR read is best-effort (degrades to `null` on staging, real values in prod).

## Domain Rating + traffic (ahref-service)

- DR **and** monthly organic traffic are owned by **ahref-service** (domain-keyed cache; it owns the Apify scrape spend). outlets-service stays cost-free — it reads ONLY ahref's CACHE endpoints, never the `*-compute` scrape endpoints.
- **discover** (`POST /orgs/outlets/discover`) fires `POST /orgs/domains/dr-compute` **non-blocking** (`.catch` + log) for the freshly-discovered domains (deduped across cycles) — a trigger, never awaited, never fails discover.
- **`GET /orgs/outlets` enriches each outlet ALWAYS-ON (no opt-in param)** with two nullable fields, keyed by the same normalized (www-stripped, lowercased) domain:
  - `domainRating` ← ahref `GET /orgs/domains/dr-status` (`latestValidDr`)
  - `trafficMonthlyAvg` ← ahref `GET /orgs/domains/traffic-history` (`trafficMonthlyAvg`)
- **The reads MUST be partial-tolerant at scale (12k+ domains).** `getDrStatusForEnrich` / `getTrafficForEnrich` (`services/ahref.ts`) chunk by URL length, run chunks at bounded concurrency (6), and tolerate failures **per chunk**: a chunk that fails its bounded transient-retry budget drops only ITS domains to `null` — never the whole set. This replaced an all-or-nothing `getDrStatusBestEffort(getDrStatus)` whose single-chunk throw nulled every outlet for a real 12k-domain brand. Do NOT reintroduce a wrap that returns `{}` on any error.
- **Chunk-level retry:** `fetchDomainCacheChunk` retries on THROWN transient transport errors (ahref cold-start / pool-saturation → `fetch failed` whose cause walks to ECONNRESET/ETIMEDOUT/ECONNREFUSED, or AbortSignal `TimeoutError`) — backoff 250/500/1000, idempotent GET so write-safe. An HTTP 5xx is a real answer (request reached ahref) and is NOT retried (fail-loud per chunk).
- **`GET /orgs/outlets/:id`** merges `domainRating` ALWAYS-ON (single domain) via `getDrStatus` (fail-loud client, route degrades to `null`). It does NOT add `trafficMonthlyAvg`.
- `null` (either field) = ahref has no cached/trustworthy value for that domain, OR that chunk stayed unreachable after retries.
- `discoverCycle` / `discoverOutletsInCategory` in `category-discovery.ts` return `{ inserted, domains }` so the route can collect discovered domains for the trigger. The `buffer/next` discovery path does NOT trigger dr-compute (follow-up, not yet wired).
- Env vars: `AHREF_SERVICE_URL`, `AHREF_SERVICE_API_KEY` (the key = ahref's own inbound key, shared).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shamanic-technologies/outlets-service](https://github.com/shamanic-technologies/outlets-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
