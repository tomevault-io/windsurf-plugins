---
trigger: always_on
description: - Canonical public paths: `/` (rankings), `/plugins` (catalog), `/plugins/:owner/:name` (plugin detail), `/docs/api` (public API reference), `/community` (community feed), `/community/p/:id` (post), `/community/about` (rules), `/account` and `/community/u/:login` (noindex).
---

# Repository instructions

## Public URL stability

- Canonical public paths: `/` (rankings), `/plugins` (catalog), `/plugins/:owner/:name` (plugin detail), `/docs/api` (public API reference), `/community` (community feed), `/community/p/:id` (post), `/community/about` (rules), `/account` and `/community/u/:login` (noindex).
- Plugin API routes use the `/api/v1/` prefix with plural resources (`/api/v1/plugins`, `/api/v1/plugins/:owner/:name`). The outward-facing search API is `https://api.deepseek1024.com/v1/plugins/search`.
- `/plugin`, `/plugin/:owner/:name`, `/packages`, `/packages/:owner/:name` and `/rankings` are permanent 301 **sources only**. Do not cite them as live URLs.
- Treat public route paths as permanent SEO contracts. Do not rename or remove them without explicit user approval and a migration plan covering permanent redirects, canonical URLs, and existing inbound links.
- Page titles, descriptions, JSON-LD and the crawlable pre-hydration shell all come from `web/worker/seo-templates.ts` and `web/worker/seo-content.ts`. Both the Worker and the React app import them; never fork the copy into a page component or a translation file.
- When replacing an already-published route, keep a permanent redirect from the old path to the canonical path.

## API backward compatibility

- Every API is a published compatibility contract, whether it is currently used by the site, a non-Web client, a third party, an authenticated tool, an internal sync job or a WebSocket client. Never assume a site-only caller updates in lockstep with the Worker.
- Within an existing API version, do not remove or rename fields, change types or nullability, reinterpret values, change defaults, status/error codes, pagination, ordering, authentication behavior or important headers. Treat new response enum values as potentially breaking. Compatible additions must remain optional or ignorable to old clients.
- Breaking changes require a new versioned route while the previous route and its behavior remain available through an explicit migration and deprecation period.
- `web/contracts/api-surface.json` is the exhaustive API inventory. Every Hono API route, Worker-owned API transport and public-host alias must be registered there. Versioned response schemas and golden semantic fixtures live beside it.
- Run `npm run test:api-contract` for every API-related change. Do not make a failure green by casually rewriting a historical schema or golden fixture; contract changes require explicit API-owner review. Keep the route coverage test, historical request behavior and known consumer-adapter tests current.

## Bound hostnames and the public API surface

The Worker answers on three custom domains, all declared in `web/wrangler.jsonc`
under `routes`, and each host has a deliberately different surface. `web/worker/public-api.ts`
is the single place that decides which is which; `web/tests/public-api.test.ts` guards it.

- `deepseek1024.com` — the website and the full `/api/...` surface, including sign-in and API-key
  management. This is the only host that serves the site.
- `www.deepseek1024.com` — a bound custom domain that exists solely to `301` to the apex host
  (`wwwRedirect`). It is not an alias you can serve content from.
- `api.deepseek1024.com` — the public developer API. It exposes an **allow-list of two paths**,
  `PUBLIC_API_PATHS` in `public-api.ts`, rewritten onto the internal routes:
  `/v1/plugins/search` → `/api/v1/plugins/search`, and `/v1/health` → `/api/v1/health`.

That host exists for third-party consumers, and its one substantive endpoint is metered
independently of the site. `/v1/plugins/search` enforces a per-caller quota — `ANONYMOUS_QUOTA`
and `AUTHENTICATED_QUOTA` in `web/worker/lib/api-quota.ts`, counters kept in D1 through
`consumeQuota`: 10/min and 50/day anonymous, 30/min and 500/day with a key. Anonymous callers are
keyed by `ip:<HMAC of CF-Connecting-IP>` so the raw address never reaches D1; authenticated callers
are keyed by `user:<id>` and not by key id, so rotating or minting keys cannot open a fresh window.
Every response carries `X-RateLimit-Daily-Limit` and `X-RateLimit-Daily-Remaining`; a rejection adds
`Retry-After` and returns `429`, with `DAILY_QUOTA_EXCEEDED` for the day window and `RATE_LIMITED`
for the minute window. `/v1/health` is deliberately unmetered.

The quota lives on the search handler in `web/worker/app.ts`, not on the host check, so
`deepseek1024.com/api/v1/plugins/search` draws down the same counters.

Four ways this gets broken, in rough order of likelihood:

1. **Assuming a 404 on `api.deepseek1024.com` is a bug.** Every path outside the allow-list returns
   `404 {"code":"NOT_FOUND"}` on purpose, and `/` returns `302` to `/docs/api`. So
   `api.deepseek1024.com/v1/registry` and `.../api/v1/registry` both 404 while
   `deepseek1024.com/api/v1/registry` works — that is the design, not a routing fault. Verify the
   host is healthy with `/v1/health`, which returns `{"status":"ok"}`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imsai-sh/dsh-1024store](https://github.com/imsai-sh/dsh-1024store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
