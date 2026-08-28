---
trigger: always_on
description: Working notes for anyone — human or agent — changing this repository. `README.md` is the front
---

# AGENTS.md

Working notes for anyone — human or agent — changing this repository. `README.md` is the front
door and `PLAN.md` holds the design reasoning; this file is the operational knowledge, including
the mistakes that already cost time.

---

## What this is

A self-hosted clone of the **WasenderAPI** HTTP interface, backed by Baileys. The defining
constraint is **strict wire fidelity**: the goal is that their published SDK runs against us
unmodified. That means reproducing inconsistencies rather than tidying them. When something in
this API looks wrong, the first question is "is this theirs?" — usually it is, and it stays.

The upstream documentation is mirrored under `docs/wasenderapi/` and is **gitignored** (their
prose is copyrighted). CI therefore skips the suites that read it. A skipped suite is honest;
a green tick on assertions that never ran is not.

---

## Commands

```bash
bun install
bun run typecheck          # every workspace with a tsconfig, derived from package.json
bun test                   # unit + contract suites
bun run contracts:generate # regenerate Zod contracts from the mirrored spec
node ops/check-dockerfile-manifests.mjs
bun  ops/check-sdk-in-sync.mjs   # bun, not node — it imports the TS contracts
```

Live integration tests need `DATABASE_URL` and self-skip without it:

```bash
set -a; . ./.env; set +a
bun test compat/integration.test.ts
```

The fidelity suite needs no phone and no production. Boot the stack against any empty database
and point it at a PAT — which is exactly what the `sandbox` CI job does:

```bash
bun run --cwd packages/db migrate
PORT=3102 bun run --cwd apps/gateway start &
PORT=3101 bun apps/api/src/index.ts &
WAPI_PAT=$(bun ops/seed-ci.ts) WAPI_BASE_URL=http://127.0.0.1:3101 bun test compat/sandbox.test.ts
```

Add `REDIS_URL` and a running `apps/webhook-worker` and the webhook-delivery test runs too — it
hosts its own sink, fabricates an inbound message and waits for the signed POST. Without Redis it
skips, because there is no worker and therefore no delivery.

Browser tests need Chromium and real Clerk keys, both one-time:

```bash
bunx playwright install chromium
clerk env pull --file apps/web/.env.local   # then add a NEXT_PUBLIC_-prefixed publishable key
bun run --cwd apps/web e2e
```

See `apps/web/e2e/README.md` — it records three sign-in approaches that fail in ways that look
like success, and is worth reading before changing anything there.

---

## Layout

| Path | Runtime | Notes |
| --- | --- | --- |
| `apps/api` | Bun + Hono | the 29 routes. **Stateless** |
| `apps/gateway` | **Node 22** | Baileys sockets. **Stateful**, internal RPC only, no public port |
| `apps/webhook-worker` | Node + BullMQ | delivery, retry, backoff, DLQ |
| `apps/web` | Next.js 16 | dashboard, guide, Clerk |
| `packages/contracts` | — | Zod contracts, response schemas, OpenAPI emit |
| `packages/core` | — | shared logic, `WhatsAppEngine` + `Storage` interfaces |
| `packages/db` | — | Drizzle schema + migrations |
| `packages/baileys-auth` | — | Postgres-backed `AuthenticationState` |
| `compat/` | — | SDK-compat, fidelity (sandbox) and live integration suites |
| `apps/web/e2e` | — | Playwright — the only thing that renders a page |
| `sdk/typescript` | — | TypeScript client. Types generated, surface hand-written |
| `sdk/python` | — | Python client. Stdlib only, same surface, snake_case |
| `sdk/go` | — | Go client. `net/http` only, nested module |
| `ops/` | — | backup, restore, retention, CI guards |

The gateway is Node, not Bun, because Baileys' WASM Signal bridge needs it. Everything else is
Bun. Do not "unify" this.

---

## The dashboard

Two halves, deliberately: an **operator console** (is this session healthy, what is it
configured to do) and a **developer workbench** (what contacts, groups, messages and webhook
deliveries exist). Everything session-scoped lives under `/sessions/{id}/…` — a contact list
without a session is meaningless, and a session picker on every page would re-express in UI
what the URL already says.

`apps/web/src/app/sessions/[id]/layout.tsx` resolves and authorises the session, so it is the
ownership check for every page beneath it.

`/audit` is the exception to the session-scoped rule and deliberately so: calls made with a PAT —
creating a session, rotating a key — belong to the account and have no session at all, so filing
the trail under a session would hide the actions most worth auditing.

**Where data comes from is a rule, not a habit:**

| | Source | Why |
| --- | --- | --- |
| Sessions, tokens | Postgres | no API equivalent that isn't PAT-scoped |
| Message log | Postgres | `message-logs` is PAT-authenticated upstream |
| Contacts, groups | our own API, internal `http://api:3001` | dogfooding: if `/api/contacts` breaks, the dashboard breaks |
| Doctor | our own API, public `https://api.wapi.crafter.run` | the edge is part of what it tests |

The web app must never hold a Personal Access Token. That is the whole reason account-scoped
reads go direct to the database rather than through HTTP.

**The doctor** (`apps/web/src/lib/doctor.ts`) has three rules that are not negotiable:

- Every call goes over the **public edge**, because "does this work end to end" includes TLS

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crafter-station/wapi](https://github.com/crafter-station/wapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
