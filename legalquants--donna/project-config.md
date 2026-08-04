---
trigger: always_on
description: This file orients a coding co-pilot — a human developer or another AI agent (Claude Code reads it
---

# CLAUDE.md — engineering guide for Donna

This file orients a coding co-pilot — a human developer or another AI agent (Claude Code reads it
automatically on session start) — so you can understand the whole project and **pick up the roadmap
from where it stands**. Read it once, top to bottom, before your first change.

For _what the product is and why_, read [docs/PRODUCT.md](docs/PRODUCT.md) first. For _how to run
it_, [README.md](README.md). This file is _how to build in it well_.

---

## 1. What Donna is, in one breath

Donna is a standalone **SvelteKit** app — a friendly, document-forward **frontend** for the
[LQ.AI](https://github.com/LegalQuants/lq-ai) legal-AI backend. Donna implements **no** legal-AI
logic itself: retrieval, the citation engine, anonymization, skills, playbooks, tabular review, and
the autonomous runtime all live in lq-ai. Donna's job is to make that power **usable, transparent,
and controllable** through a clean reading-first UI.

Donna talks to lq-ai **only through its published API**, and **vendors** that backend as a pinned
git submodule (`vendor/lq-ai`) so the whole product runs from one compose file.

## 2. The cardinal rules (violate these and you'll break the project's model)

1. **Never edit `vendor/lq-ai`.** It is a pinned submodule, not our code. If you need backend
   behavior that doesn't exist, that's an **upstream request** (§8), not a local patch.
2. **Consume the contract; never hand-fork it.** API types are generated from lq-ai's OpenAPI into
   `src/lib/api/backend.d.ts` via `npm run gen:api`. Derive types from there. (Where the backend
   types a field loosely — `additionalProperties` — hand-type it in a small parser and say so in a
   comment; see the `parseTabularResults` / `parseFindingList` precedents.)
3. **The bar is green, not "no worse."** `npm run check` = 0 errors / 0 warnings; `npm run lint` =
   prettier + eslint fully clean; the unit suite passes. Keep it that way.
4. **Merge PRs with a MERGE COMMIT.** A squash would orphan the two one-time-format SHAs in
   `.git-blame-ignore-revs`. Never squash to `main`.
5. **Evidence before claims.** "It works" requires a run you can point to — a passing test, a live
   e2e, an actual page load. Report failures faithfully.

## 3. Architecture — the backend-for-frontend (BFF)

The browser talks **only** to Donna's SvelteKit server. That server:

- Holds the lq-ai JWT **access + refresh tokens in httpOnly cookies** (never exposed to client JS).
- Attaches `Authorization: Bearer` when proxying to the lq-ai `api`, and **transparently refreshes
  on `401`**. No CORS anywhere.
- Is the single trust boundary: auth lives in `src/hooks.server.ts` + `src/lib/server/`.

Consequences you must internalize:

- **Server-only code** (cookies, the authed `lqClient`, auth wrappers) lives under
  `src/lib/server/`. Never import it into client code.
- A page gets backend data through a **SvelteKit `load`** (SSR) and mutates through **form actions**
  or small **BFF proxy routes** (`+server.ts`) — not by calling lq-ai from the browser.
- Proxy routes exist to (a) attach auth and (b) avoid page/endpoint route collisions — e.g.
  `/prompts/items` sits beside the `/prompts` page, and the `/tabular-executions/[id]` proxy is a
  separate top-level group precisely so it doesn't collide with the `/tabular/[executionId]` page.

## 4. Repo layout

```
src/                         SvelteKit app
  routes/(app)/              authed app routes (the product)
  routes/(auth)/             login / change-password (guarded by hooks.server.ts)
  lib/                       feature modules — one dir per domain
    server/                  SERVER-ONLY: session cookies, authed lqClient, auth wrappers
    api/                     generated OpenAPI types (npm run gen:api) — do not hand-edit
    docpanel/                document panel: PDF render, highlight, TextViewer (md/plain)
    automations/             autonomous runs: sessions, receipts, results, schedules, watches
    tabular/ playbooks/ skills/ prompts/ matters/ knowledge/ inference/ …
  hooks.server.ts            auth routing / token refresh (the global guard)
vendor/lq-ai/                pinned lq-ai backend (submodule) — NEVER edit
docs/                        see docs/README.md for the full index
tests/                       Playwright e2e (live, against the running stack)
static/learn/                interactive playgrounds served by the /about guide
```

## 5. Dev stack — run & verify

Prereqs: Docker + Compose v2, Node 22+. Full setup in the README; the essentials:

```bash
# cold start (loads .env, builds, brings up the explicit service list)
set -a; . ./.env; set +a
docker compose up -d --build postgres redis minio gateway api donna-web ingest-worker arq-worker
```

- App at **http://localhost:13002**, lq-ai api at **http://localhost:18000** (ports are _shifted_
  in `.env` so Donna can coexist with a separate raw lq-ai dev stack on the defaults).
- `api` is the **single schema migrator** (workers run with `LQ_AI_SKIP_MIGRATIONS=1`). After a pin
  bump, rebuild `api` + `arq-worker` + `ingest-worker` together so siblings don't crash-loop on a
  revision mismatch.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LegalQuants/Donna](https://github.com/LegalQuants/Donna) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
