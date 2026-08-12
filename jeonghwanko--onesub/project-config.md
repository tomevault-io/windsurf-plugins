---
trigger: always_on
description: This is the canonical repository guide for coding agents. `CLAUDE.md` imports this file so Codex
---

# OneSub Repository Guide

This is the canonical repository guide for coding agents. `CLAUDE.md` imports this file so Codex
and Claude share one set of project instructions. Keep durable project knowledge here; keep public
setup and API documentation in `README.md` and `docs/`.

**How to use this guide.** Read *Build Model and Traps* before your first edit — it is the section
that prevents silent failures. Then jump: *Source Map* for where code lives, *Start Here by Task* for
which files a given task touches, *Contract Change Checklist* for changes that must move several
files at once, *Change Workflow* and *Before You Call a Task Done* for what to run and report.
`docs/AI-WORKFLOW.md` holds copy-ready prompts; this file holds the rules.

## Project Scope

OneSub is a self-hosted in-app purchase backend and client toolkit. It validates Apple StoreKit 2
and Google Play receipts, processes subscription webhooks, stores subscription and one-time purchase
state, exposes entitlement/admin/metrics APIs, and provides React Native and Unity clients.

This public repository is the MIT-licensed Core source of truth. Commercial Unity Editor automation
and MCP for Unity custom tools live in the separate private `onesub-unity-pro` repository. Do not
copy Pro sources into this repository. See `docs/UNITY-PRO.md` for the compatibility boundary.

## Repository Map

| Path | Role |
|---|---|
| `packages/shared` | `@onesub/shared`: canonical cross-package types, status values, error codes, and route constants |
| `packages/providers` | `@onesub/providers`: dependency-free App Store Connect and Google Play product-management wrappers |
| `packages/server` | `@onesub/server`: Express middleware/server, receipt validation, webhooks, stores, admin APIs, metrics, OpenAPI, and tracing |
| `packages/sdk` | `@jeonghwanko/onesub-sdk`: React Native provider, hook, paywall components, and HTTP client |
| `packages/mcp-server` | `@onesub/mcp-server`: stdio MCP tools for setup, product management, diagnostics, and simulation |
| `packages/cli` | `@onesub/cli`: `onesub init` scaffolder, server templates, and the `onesub dev` fully mocked server used for local and agent testing |
| `packages/dashboard` | Private npm workspace for the self-hosted Next.js operations dashboard; shipped as a Docker image |
| `packages/unity` | `com.onesub.unity`: public Unity 2022.3+ purchasing and server-validation Core package |
| `packages/unity-platform-services` | Optional Unity sharing, review, leaderboard, and authentication helpers; not part of purchasing Core |
| `examples` | Runnable server and Expo examples. Not npm workspaces, but inside this checkout they still resolve `@onesub/server` through the root `node_modules` symlink to `packages/server` — so they do exercise your local build. The version pin in their own `package.json` only applies to a standalone copy |
| `bench` | k6 status/webhook load tests, run by the scheduled `bench` workflow |
| `scripts` | `validate-docs.mjs`, which backs `npm run docs:check` |
| `docs` | Architecture, security, deployment, migration, receipt-error, and Unity boundary documentation |

The two Unity packages are UPM packages, not npm workspaces. `validate-unity-packages.ps1` lives at
the repository root, not under `scripts`.

## Source Map

File-level orientation, so you can open the right file instead of grepping for it. Paths are stable;
verify contents against the source before quoting them.

| Area | Files |
|---|---|
| Route constants, statuses, error codes | `packages/shared/src/constants.ts` |
| Cross-package types (config, `SubscriptionInfo`, `PurchaseInfo`) | `packages/shared/src/types.ts` |
| Middleware assembly and public exports | `packages/server/src/index.ts` |
| Routes | `packages/server/src/routes/` — `validate.ts`, `status.ts`, `purchase.ts`, `admin.ts`, `entitlements.ts`, `metrics.ts`, `apple-offer.ts`, `webhook-apple.ts`, `webhook-google.ts`, `webhook.ts` |
| Admin-secret comparison | `packages/server/src/routes/secret-compare.ts` |
| Sandbox-only entitlement overrides (process-local, never persisted) | `packages/server/src/test-overrides.ts` |
| Providers (Apple, Google, mock) | `packages/server/src/providers/` |
| Stores | `packages/server/src/store.ts` (in-memory), `packages/server/src/stores/postgres.ts`, `stores/redis.ts`, `stores/schema.ts` (DDL constants) |
| SQL schema shipped to users | `packages/server/sql/schema.sql` (parity-tested against `stores/schema.ts`) |
| OpenAPI spec | `packages/server/src/openapi.ts` (parity-tested against mounted routers) |
| Webhook durability | `packages/server/src/webhook-queue.ts`, `webhook-events.ts` |
| Outbound HTTP, caching, logging, tracing | `packages/server/src/http.ts`, `cache.ts`, `logger.ts`, `log-format.ts`, `tracing.ts` |
| Multi-app credential resolution | `packages/server/src/apps.ts` |
| SDK provider (listeners, drain gate, purchase/restore entry points) | `packages/sdk/src/OneSubProvider.tsx` |
| SDK pure purchase-flow logic (in-flight map, native error mapping, type resolution) | `packages/sdk/src/purchaseFlow.ts` |
| SDK request shaping (Google offer tokens, platform args) | `packages/sdk/src/iapRequest.ts` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeonghwanko/onesub](https://github.com/jeonghwanko/onesub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
