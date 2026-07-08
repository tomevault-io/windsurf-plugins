---
trigger: always_on
description: This repository contains the Horojob backend API (Fastify + TypeScript + MongoDB).
---

# AGENTS.md

## Purpose

This repository contains the Horojob backend API (Fastify + TypeScript + MongoDB).

Mobile client lives in:

`../horojob`

Primary responsibilities in this repo:

- implement and maintain HTTP endpoints
- validate external input
- enforce business rules
- persist data and indexes in MongoDB
- keep contracts stable for the mobile app

---

## Current project layout

`horojob-server/`

- `src/`
  - `server.ts` (startup + schedulers + graceful shutdown)
  - `app.ts` (Fastify app composition + route registration)
  - `config/env.ts` (runtime env parsing and defaults)
  - `db/mongo.ts` (collection typing + index creation)
  - `routes/`
    - `auth.ts`
    - `astrology.ts`
    - `billing.ts`
    - `cities.ts`
    - `health.ts`
    - `jobs.ts` (`jobs.handlers.ts` implementation)
    - `notifications.ts`
  - `services/` (domain logic + integrations + schedulers)
- `docs/`
  - `auth-and-session-contract.md`
  - `backend-api-runtime-map.md`
  - `jobs-api-contract.md`
  - `mongo-collections-and-indexes.md`
  - `documentation-audit-2026-03-29.md`
  - `morning-briefing-api.md`
  - `notifications-and-billing-contracts.md`
  - `redis-cache-and-locks-current-state.md`
  - `redis-cache-plan.md`
  - `skills-usage-log.md`
- `skills/`
  - `node-api/SKILL.md`
  - `mongo-contracts/SKILL.md`
- `README.md`

Current architecture reality:

- no dedicated `src/controllers` directory
- no dedicated `src/repositories` directory
- no dedicated `src/models` directory
- no dedicated `src/middleware` directory

This repo is separate from `../horojob`; it is not a monorepo.

---

## Runtime and scripts

- Node engine: `>=22 <23`
- Available scripts:
  - `npm run dev`
  - `npm run build`
  - `npm run lint`
  - `npm run check:app`
  - `npm run check:scripts`
  - `npm run start`
  - `npm run check`
  - `npm test`
  - `npm run verify`
  - `npm run smoke:routes`
  - `npm run ci:smoke`
- Parser/browser fallback work may require:
  - `npx playwright install chromium`

---

## Local skills routing

Project-local skills in this repo:

- `skills/node-api/SKILL.md`
- `skills/mongo-contracts/SKILL.md`
- `skills/mobile-server-contract-sync/SKILL.md`

Routing rules:

- use `node-api` for endpoint behavior, request validation, response shaping, auth checks, and service-level business logic
- use `mongo-contracts` when a task touches Mongo collections, indexes, query shapes, aggregation, TTL, or compatibility with existing documents
- use `mobile-server-contract-sync` when a task spans this repo and `../horojob` together, especially `src/routes/*`, `../horojob/src/services/*`, DTO parsing, contract docs, or smoke checklists that must stay aligned across both repos
- if `mobile-server-contract-sync` and `node-api` both apply: start with `mobile-server-contract-sync` for the shared contract boundary, then `node-api` for server-side transport behavior
- if both apply in one task: run `node-api` first for transport contract, then `mongo-contracts` for persistence safety/performance

Coordination with system-level skills:

- use `mvp-code-review` for review/audit tasks when that skill is available in the session
- if system-level skills are unavailable, continue with local skills and repository docs
- for implementation specifics in this repo, local skills take precedence over generic guidance when rules conflict

---

## Skill usage tracking

- For each non-trivial task, append one row to `docs/skills-usage-log.md`.
- Minimum columns: date, task summary, primary skill, secondary skills, outcome.
- Keep this log as the source for periodic routing and effectiveness review.

---

## Source of truth order

1. explicit user instructions
2. this file
3. focused docs:
   - `docs/auth-and-session-contract.md`
   - `docs/backend-api-runtime-map.md`
   - `docs/jobs-api-contract.md`
   - `docs/mongo-collections-and-indexes.md`
   - `docs/notifications-and-billing-contracts.md`
   - `docs/redis-cache-and-locks-current-state.md`
   - `docs/morning-briefing-api.md`
4. `README.md` (runbook + endpoint list)
5. endpoint contracts in `src/routes/*.ts` and `src/services/astrology/*Routes.ts`
6. shared business logic in `src/services/*.ts`
7. collection shapes/indexes in `src/db/mongo.ts`
8. tests in `src/services/*.test.ts` and `src/routes/*.test.ts`

If docs and code conflict, prefer shipped route contracts and current DB/index behavior.

---

## Architecture reality (current)

- `app.ts` registers top-level route groups:
  - `/api/auth`
  - `/api/cities`
  - `/api/astrology`
  - `/api/jobs`
  - `/api/billing`
  - `/api/notifications`
- Route files usually include:
  - request parsing/validation (`zod`)
  - auth checks
  - orchestration and response mapping
- Service files hold reusable domain logic and external-provider integrations.
- `src/db/mongo.ts` is the central place for collection typings and index setup.
- Schedulers are started from `server.ts`:
  - daily transit
  - job metrics alerts
  - burnout alerts
  - interview strategy refill

---

## Change rules

Prefer:

- minimal, targeted diffs
- preserving response shape/status codes for existing endpoints
- reusing existing service/db helpers
- additive changes over breaking replacements

Avoid:

- silent API contract breaks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [p1vzz/horojov-server](https://github.com/p1vzz/horojov-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
