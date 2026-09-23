---
trigger: always_on
description: Guidance for AI coding agents working in the Floci UI repository.
---

Guidance for AI coding agents working in the Floci UI repository.

This file defines repository-specific operating rules for autonomous or semi-autonomous
coding agents. Follow these instructions unless a maintainer explicitly tells you otherwise.

`AGENTS.md` is the canonical agent-instructions file for this repository, following the
[AGENTS.md standard](https://agents.md/). `CLAUDE.md`, `GEMINI.md`, and
`.github/copilot-instructions.md` are symlinks to this file — edit `AGENTS.md` only.

---

## Project Overview

Floci UI is the web console / DevTools for [Floci](https://floci.io), the local multi-cloud
emulator. It is an AWS-Console-style UI for a locally running cloud runtime.

It does **not** emulate anything itself. The frontend renders cloud resources; the API
translates the UI's REST/JSON requests into cloud-SDK calls against the locally running
Floci emulators (AWS, Azure, GCP).

- pnpm workspace monorepo, two packages:
  - `packages/frontend` — React + Vite + TypeScript, served on port `4500`
  - `packages/api` — Bun + Hono + AWS SDK v3, served on port `4501`
- Emulator endpoints it talks to: Floci core (AWS) `:4566`, Floci-AZ `:4577`, Floci-GCP `:4588`

---

## First Principles

When making changes, follow these priorities:

1. Use real cloud-provider contracts — never invent custom backend endpoints for UI convenience
2. Reuse the schema-driven multi-cloud pattern instead of bespoke per-service code
3. Keep the frontend talking only to `/api/*`; never reach a cloud endpoint directly from the browser
4. Prefer real empty states over fake/sample data
5. Keep changes narrow and testable

Critical rules:

- Do not add custom protocols just for the UI unless the core project accepts that contract
- Do not have the frontend call AWS/Azure/GCP endpoints directly — always go through `packages/api`
- Do not introduce decorative data or fake operational metrics — unwired states stay empty
- Do not perform broad refactors unless the task explicitly requires them

---

## Architecture

```
Browser (React/Vite :4500)
  → /api/*  (Hono, Bun :4501)
    → CloudProxyService → CloudAdapterRegistry → CloudServiceAdapter
      → AWS SDK v3 (:4566) | Floci-AZ HTTP (:4577) | Floci-GCP HTTP (:4588)
```

The repo is mid-migration from an older **AWS-only per-service** style to a newer
**schema-driven, multi-cloud generic explorer**. The generic pattern is the one to use
for all new work; the legacy routes survive only for deep EC2 panels and Secrets Manager.

### The multi-cloud SPI (the part you will use most)

- `packages/api/src/cloud-spi/serviceCatalog.ts` — **the single source of truth for which
  services exist.** `CloudServiceType` derives from its keys; nav metadata (display name,
  icon hint, group, route) is served to the frontend from here.
- `packages/api/src/cloud-spi/types.ts` — `CloudProvider` (`aws|azure|gcp`), the
  `CloudServiceAdapter` interface, `ServiceSchema`, and the status shapes.
- `packages/api/src/cloud-spi/errors.ts` — the typed errors adapters throw; mapped to HTTP
  once in `routes/clouds.ts` (with `adapter-aws/awsErrors.ts` for SDK failures).
- `packages/api/src/registry/CloudAdapterRegistry.ts` — registry keyed by `"cloud:service"`.
  Availability is derived from it, so registering an adapter is what lights up the nav.
- `packages/api/src/service/CloudProxyService.ts` — the single dispatcher.
- `packages/api/src/service/runtimeProbe.ts` — per-runtime liveness probes.
- `packages/api/src/cloudProxy.ts` — where adapters are instantiated and registered.
- `packages/api/src/routes/clouds.ts` — the generic `/api/clouds/...` REST surface.
- `packages/api/src/cloudProxy.test.ts` — guards that no schema advertises a capability its
  adapter cannot perform.

A `ServiceSchema` (fields, `actions`, `capabilities`, `filters`, `columns`) drives the UI:
the frontend's `DynamicResourceView` renders list / create / delete / inspect generically
from the schema — most services need **no bespoke UI**.

### Frontend layout

- `packages/frontend/src/App.tsx` — routes (`/console/:cloud`, `/cloud-explorer/:cloud/:service`)
- `packages/frontend/src/components/Layout.tsx` — nav, rendered from `GET /clouds/:cloud/services`
- `packages/frontend/src/api/queries/cloudQueries.ts` — shared cloud/service/status queries
- `packages/frontend/src/components/serviceIcons.ts` — `iconKey` -> component, with a fallback
- `packages/frontend/src/components/DynamicResourceView.tsx` — schema → table/form/inspector orchestrator
- Reusable: `ResourceTable`, `DynamicFormRenderer`, `ResourceInspector`, `StorageObjectBrowser`,
  `CosmosNoSqlPanel`, `EmptyState`, `lib/capabilities.ts`
- API client: `src/api/cloudProxyClient.ts`, `src/api/api.ts`, `src/api/HttpClient.ts`

### Legacy (do not extend without reason)

`packages/api/src/routes/{ec2,rds,eks,secretsmanager}.ts` and the matching
`features/ec2/*` frontend code. New services go through the generic SPI, not here.

---

## Build & Run

    pnpm install
    pnpm dev          # API (:4501) + frontend (:4500) together
    pnpm dev:api      # API only
    pnpm dev:web      # frontend only

Requires a running Floci core (`:4566`) — see `README.md` / `docker compose` (use the
`multicloud` profile to also start Azure + GCP).

### Checks (run all before finishing)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [floci-io/floci-ui](https://github.com/floci-io/floci-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
