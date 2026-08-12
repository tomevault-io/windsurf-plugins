---
trigger: always_on
description: Guidance for AI agents working on `@agentcash/router`.
---

# AGENTS.md

Guidance for AI agents working on `@agentcash/router`.

## What this is

A protocol-agnostic route framework with a framework-agnostic core: routes are Web-standard `Request`/`Response` handlers dispatched through an embedded Hono app. Hosts on Next.js (per-file, or a single catch-all via `@agentcash/router/next`), Hono, Bun, or any fetch runtime through `router.fetch` / `router.hono()`. Provides x402 payments, MPP payments, SIWX identity auth, and API key auth behind a single fluent builder. A route definition is 3 to 6 lines; everything else (pricing, discovery, OpenAPI, settlement) is derived.

## Guiding principles

1. Route definition is 3 to 6 lines.
2. Single source of truth: the route registry drives discovery, OpenAPI, and pricing.
3. Pricing modes (`.paid()`, `.upTo()`, `.session()`) and identity auth (`.siwx()`, `.apiKey()`) compose; `.unprotected()` opts out. Exactly one pricing mode per route.
4. Observability is pluggable via `RouterPlugin`. No boilerplate.
5. The package owns x402 and MPP server lifecycles (init, verify, settle).
6. Compose, do not reimplement. Delegate to `@x402/*`, `@coinbase/x402`, and `mppx`.

## Layout

```
src/
  index.ts              public surface — createRouter / createRouterFromEnv
  builder.ts            fluent RouteBuilder (.paid / .upTo / .session / .siwx / .apiKey / .unprotected)
  registry.ts           Map-backed route registry
  constants.ts          network ids, USDC asset/decimals, default facilitator
  types.ts              core types (RouteEntry, HandlerContext, HttpError, PaidOptions, UpToOptions, SessionOptions)
  plugin/               RouterPlugin types + lifecycle dispatch
  init/                 protocol init (x402.ts, x402-server.ts, mpp.ts, mppx.ts)
  protocols/            x402/ and mpp/ strategies, detect.ts, accepts
  auth/                 siwx.ts, api-key.ts, normalize-wallet.ts
  kv-store/             one KvStore backs siwx nonce, siwx entitlement, mpp replay
  pricing/              fixed, tiered, dynamic (args-derived), upto-charge, metered-charge, format (atomic conversion)
  pipeline/             orchestrate.ts + steps/ + flows/ (paid → static-paid | dynamic-paid; siwx-only, api-key-only, unprotected)
  discovery/            openapi, llms-txt, well-known (deprecated surface), not-found
  config/               RouterConfig + env schema (single source of truth), RouterConfigError, issue codes
```

## Pipeline

`auth check -> body parse -> validate -> 402 challenge -> payment verify -> handler -> settle -> finalize`

The `body parse -> validate -> 402 challenge` prefix only holds when the challenge needs the body: args-derived/tiered pricing, `.validate()`, or a checkout session (`shouldParseBodyEarly`). On other paid routes a bare unpaid probe gets its 402 without the body being inspected (malformed body ⇒ 402, not 400); the paying retry parses and validates before verify/settle, so a 400 never charges the caller.

Paid `.paid()` routes may set `mpp: { settleBeforeHandler: true }` (or chain `.mpp({ settleBeforeHandler: true })` on auto-priced routes) to broadcast MPP transaction (pull) credentials at verify instead of after the handler. x402 on the same route is unaffected. MPP hash (push) credentials already settle at verify. Use `.settlement({ onSettledHandlerError })` when eager MPP settle can charge before an upstream failure.

## Naming

Two constructor-verb families, split by audience:

- `create<Noun>` — public, top-level factories that return long-lived objects: `createRouter`, `createRouterFromEnv`, `createRequestHandler`, `createUptoChargeContext`.
- `build<Noun>` — internal, per-request assembly of protocol payloads and contexts: one verb, one domain noun, named after the domain concept, never after an HTTP status code or transport detail (the function that builds a payment challenge is `buildChallengeResponse`, not `build402`). The challenge family shares the `Challenge` backbone: `buildChallengeResponse`, `buildChallengeExtensions`, `buildSiwxChallenge`, `buildSessionChallenge`, `buildX402Challenge`.

Pipeline steps use `run<Noun>` (executes and may answer the request), `resolve<Noun>` (computes a value), or `try<Noun>` (optional fast path returning null to continue).

## Critical rules

- **Error handling.** Respect `.status` on any thrown error, not just `HttpError`. Pattern: `throw Object.assign(new Error('msg'), { status: 409 })`. Three typed error classes, by phase: `RouterConfigError` (router construction), `RouteDefinitionError` (route registration, thrown at module-import time — never a JSON response), `HttpError` (request time, becomes a structured JSON response). New registration-time guards must throw `RouteDefinitionError`, not plain `Error`.
- **SIWX challenge.** Must return an x402v2 challenge with a `PAYMENT-REQUIRED` header and a JSON body whose `extensions['sign-in-with-x']` carries `info` (an object with `domain`, `uri`, `version`, `chainId`, `type`, `nonce`, `issuedAt`, `expirationTime`, `statement`), `supportedChains`, and an optional `schema`. The header-encoded challenge and the JSON body must stay identical.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Merit-Systems/agentcash-router](https://github.com/Merit-Systems/agentcash-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
