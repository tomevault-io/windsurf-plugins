---
trigger: always_on
description: Every package is `@chia/<name>` and depends on siblings with `workspace:*`. Its `exports` map is the authoritative module list — read it rather than expecting this file to enumerate modules.
---

# Packages

Every package is `@chia/<name>` and depends on siblings with `workspace:*`. Its `exports` map is the authoritative module list — read it rather than expecting this file to enumerate modules.

## API architecture

Three packages carry it: `api` (contracts, handlers, guards, ports), `service-kit` (context, policies, errors, transport adapters) and `db` (schemas, repositories).

**Contract-first oRPC.** The wire contract lives in `packages/api/orpc/contracts/*.contract.ts` and is composed in `router.contract.ts`. Handlers live in `packages/api/orpc/routes/*.route.ts` and are composed in `router.ts` via `contractOS` (`implement(routerContract)`). The two trees must stay key-for-key identical. Consumers import the contract type only.

**Guards and policies.** Authorization logic lives once as a _policy_ in `packages/service-kit/src/policies` (`sessionPolicy`, `apiKeyPolicy`, `adminPolicy`, `callerPolicy`, `rateLimitPolicy`, `captchaPolicy`, `aiKeyPolicy`) and is bound to each transport by a thin adapter: `runPolicy` for oRPC middleware (`packages/api/orpc/guards/`), `toHonoMiddleware` for Hono middleware (`apps/service/src/guards/`). Write new authorization as a policy, and let the guard stay a binding.

**Context injection.** `packages/api` parses no env and holds no module-level state. Everything the guards and routes need from the host travels on the oRPC context, `BaseOSContext` in `packages/api/orpc/utils.ts`:

- `config` — required: rate-limit budget, project id, AI key material.
- `hooks.onFeedChanged` / `hooks.onFeedRemoved` — optional feed lifecycle hooks (`FeedHooks`), fired by the content write paths.
- `indexing` — optional `IndexingService` port; starts and reconciles resource index runs.
- `agentKinds` — optional map of `AgentKindService` keyed by `agent_session.kind`.

The port interfaces live in `packages/api/orpc/services/` (`agent.service.ts`, `indexing.service.ts`) next to `requireIndexing(context)` / `requireAgentKind(context, kind)`, which answer `SERVICE_UNAVAILABLE` when the context lacks the port. `apps/service` is the only process that runs the router and supplies all of these in `createORPCContext`. Anything that needs a long-lived process, a DB handle or gateway credentials belongs in the app, not here.

**Data access.** oRPC handlers never write raw Drizzle; they call repositories exported as `@chia/db/repos/*`. Write logic shared with workflow steps lives in `packages/api/<domain>/write` (today `feeds/write.ts`) and takes its `FeedHooks` as an explicit argument, so a durable turn can call it with no request to authorize against.

**Errors.** `AppError` codes mirror oRPC's common codes, so a policy failure maps onto `errors[code]()` without translation.

## Package guide

### `api`

`orpc/` — `contracts/`, `routes/`, `guards/`, `services/` (the ports), `router.contract.ts`, `router.ts`, `utils.ts` (`BaseOSContext`, `contractOS`, `baseOS`). Domain modules beside it: `feeds/` (search, access, write), `resources/` (the RAG resource registry and adapters — see [`docs/rag-architecture.md`](../docs/rag-architecture.md)), and external clients each with their own env: `github`, `spotify`, `s3`, `email` (Resend), `betterstack`, `captcha`. `services/env.ts` holds the service-endpoint env the frontends compose.

### `service-kit`

What every service app boots with. `bootstrap.ts` — `createServiceFactory()` (populates `ServiceContext` on each request: headers, client IP, `db`, `kv`, `auth`) and `bootstrap(app)` (logger, Sentry, error handler, body-size cap, CORS, maintenance). `context.ts` — `ServiceContext`, the per-request shape shared by Hono (`c.var`) and oRPC. Plus `policies/`, `adapters/{hono,orpc}.ts`, `errors.ts` (`AppError`, `APP_ERROR_STATUS`), `middlewares/`.

### `db`

Drizzle 1.0 on Postgres. `src/schemas/` — tables and relations, aggregated by `schema.ts` (`./schema`) for Drizzle and Better Auth; `src/libs/` — repositories, exported as `./repos/<domain>`. `src/client.ts` — `connectDatabase(env, { withCache })`, memoized per URL **and** cache setting; the request path uses the `DrizzleCache` (Redis, explicit `$withCache` only), workflow steps ask for `withCache: false`. `src/types.ts` — pure enums (`Locale`, `FeedType`, `Role`, …) safe to import anywhere. Migrations in `.drizzle/`.

### `auth`

Better Auth. `base-auth.ts` — the options (magic link, passkey, API key, admin, organization plugins; `resend` and the email template are imported lazily). `server.ts` — `createAuth(db, kv)`, memoized to one instance per process. `client.ts` / `client.rsc.ts` — the browser and RSC clients; `utils.ts` — cookie helpers and the `X-CH-API-KEY` header name.

### `kv`

Keyv over Redis (also Valkey, Postgres, Upstash adapters). `adapters/redis.ts` exposes `getRedisKv()`, the memoized singleton the request context carries; `drizzle/cache.ts` is the `DrizzleCache` `db` plugs in; `upstash/with-rate-limiter` wraps Upstash's limiter.

### `ai`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Chia1104/chia1104.dev](https://github.com/Chia1104/chia1104.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
