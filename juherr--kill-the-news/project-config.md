---
trigger: always_on
description: This file provides guidance to Claude Code when working in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## Commands

```bash
npm install           # Install dependencies (also builds client scripts via prepare)
npm run dev           # Start local dev server (wrangler dev)
npm test              # Run all tests once
npm run test:watch    # Run tests in watch mode
npm run test:coverage # Run tests with coverage report
npm run build         # Dry-run deploy bundle (wrangler deploy --dry-run)
npm run build:client  # Compile client scripts only (src/scripts/client → src/scripts/generated)
npm run deploy        # Deploy to Cloudflare production
npm run format        # Format with Prettier
```

Run a single test file:

```bash
npx vitest run src/routes/admin.test.ts
```

## Project summary

kill-the-news is a Cloudflare Worker that ingests email newsletters and exposes them as private RSS/Atom feeds. Self-hosted, free-tier-friendly (Cloudflare + ForwardEmail).

## Development approach

Work **test-first (TDD)** and **domain-driven (DDD)** in this repo — both are first-class, not optional.

**TDD.** Write or extend a test before/with the change, then make it pass. Mirror the existing test layout (`*.test.ts` next to the source, `createMockEnv()` from `src/test/setup.ts`, MSW for outbound HTTP). End every change green: `npx tsc --noEmit`, `npm test`, and `npm run build` (dry-run deploy) must all pass before declaring done.

**DDD.** Before adding logic, check whether the domain already models the concept — reach for the value objects in `src/domain/value-objects/` (`EmailAddress`, `Domain`, `FeedId`, `MailboxId`, `Lifetime`, `SenderPolicy`) and the `Feed` aggregate rather than re-deriving things ad hoc. New behavior belongs on the type that owns the data (e.g. "sender site URL" lives on `EmailAddress`, not in a helper). Respect the layering and aggregate rules below — imports point inward (routes → application → domain; infrastructure implements ports), and never reach across a layer for convenience (e.g. importing a favicon/infra helper just to parse a domain). When the same derivation appears twice, that's the signal to push it onto a domain type.

## Architecture

Single Cloudflare Worker built with Hono. Routes:

| Method                               | Path                                                                   | Purpose |
| ------------------------------------ | ---------------------------------------------------------------------- | ------- |
| `GET /`                              | Public status page (monitoring counters + link to admin)               |
| `POST /api/inbound`                  | Webhook from ForwardEmail; IP-allowlisted to their MX sources          |
| `/api/v1/feeds*`                     | Versioned REST API (Bearer/proxy auth) — feeds + emails CRUD           |
| `GET /api/v1/stats`                  | Public monitoring counters (JSON, CORS); canonical stats endpoint      |
| `GET /api/openapi.json`              | OpenAPI 3.1 spec (public)                                              |
| `GET /api/docs`                      | Rendered API reference (Scalar, public)                                |
| `GET /rss/:feedId`                   | Public RSS 2.0 feed (conditional GET: ETag/Last-Modified/304)          |
| `GET /atom/:feedId`                  | Public Atom feed (WebSub hub header; conditional GET ETag/304)         |
| `GET /json/:feedId`                  | Public JSON Feed                                                       |
| `GET /entries/:feedId/:entryId`      | Individual email HTML view                                             |
| `GET /files/:attachmentId/:filename` | R2 attachment serving                                                  |
| `GET /admin`                         | Password-protected admin UI                                            |
| `GET /admin/opml`                    | OPML export of all feeds (admin-protected)                             |
| `/hub`                               | WebSub hub (subscribe/publish)                                         |
| `GET /favicon.svg`, `/favicon.ico`   | Project favicon (envelope logo); fallback for per-feed favicons        |
| `GET /favicon/:feedId`               | Per-feed favicon from the last sender's domain (falls back to project) |
| `GET /health`                        | Health check                                                           |
| `email`                              | Cloudflare Email routing handler (alternative to ForwardEmail webhook) |

### Source layout

```
src/
  index.ts                  # App entrypoint: CORS, IP middleware, route mounting, email handler export
  config/constants.ts       # Shared constants (TTLs, limits)
  types/index.ts            # Env, FeedConfig, EmailData, WebSubSubscription, etc.
  domain/                   # Framework-agnostic core (no Hono/infra imports leak out)
    feed.aggregate.ts       # Feed aggregate: consistency boundary; holds domain FeedState (camelCase), exposes intention-revealing reads, never raw state/metadata
    feed-state.ts           # FeedState: the aggregate's config in domain (camelCase) vocabulary — NOT the snake_case persistence DTO

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juherr/kill-the-news](https://github.com/juherr/kill-the-news) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
