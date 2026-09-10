---
trigger: always_on
description: Kytelink is developed agent-first: AI agents boot the product, log in with
---

# CLAUDE.md

Kytelink is developed agent-first: AI agents boot the product, log in with
known credentials, and actually use it. If you're an agent landing in this
repo cold, you should be driving the product within a minute — read this
section first.

## Run the product as an agent

```bash
pnpm install
pnpm agents
```

That's the whole bootstrap: on a fresh clone `pnpm agents` runs the setup
wizard non-interactively (`pnpm run setup --all` — writes `.env` with fresh
secrets, starts the full Docker stack, migrates), then seeds the agent
accounts, then boots every app with `AGENT_MODE=true` on **dev port + 1000**,
so an agent session never collides with a human dev's `pnpm dev` session on
the same machine:

| App | Dev port | Agent port |
| --- | --- | --- |
| web (editor + public profile) | 3000 | **4000** |
| landing (kytelink.com marketing) | 3001 | **4001** |
| admin | 3002 | **4002** |
| api (Fastify + tRPC) | 3003 | **4003** |
| local CDN (shared, not shifted) | 5003 | 5003 |

**Logins** (OTP is always `000000` in agent mode, for any `*@kytelink.dev`
address):

- `agent@kytelink.dev` — personal org, one published `@agent` kyte, one
  unpublished draft, MANAGER in the seeded agency org (`org_agency_demo`).
- `agent-admin@kytelink.dev` — platform `ADMIN`, full admin app access.

Fast path for scripted flows: `POST {api}/auth/dev-login {"email":"..."}`
mints a real session and sets the signed cookie in one call — skips the
login screen entirely.

**One-line tour:** web editor on :4000, admin on :4002 — log in as
`agent@kytelink.dev`, OTP `000000`.

Both login mechanisms exist ONLY when `AGENT_MODE=true`, and `apps/api`
**refuses to boot** if `AGENT_MODE=true && NODE_ENV=production`. Full detail:
`git show faa5f4d^:rewrite/24-agents.md`.

**Stack lifecycle:** `pnpm dev` and `pnpm agents` each write a lockfile
(`.dev-dev.lock` / `.dev-agents.lock`) recording their process tree. A new run
first reaps whatever a dead session left behind, and refuses to start while
the previous orchestrator is still alive — `pnpm stop` tears down both stacks
from anywhere. Never start app processes directly (`pnpm --filter ... dev`,
raw `next dev`/`tsx watch`); processes started outside the orchestrators are
invisible to the reaper and will leak when the session dies.

## Design system — read before touching any UI

[`design/DESIGN-SYSTEM.md`](./design/DESIGN-SYSTEM.md) is **gospel** for all
visual work across landing, web, and admin. It distills the approved brand
prototype archived at `design/handoff/kytelink-redesign-prototype.html`. Do
not invent colors, radii, shadows, or typography — look them up there. Where
older design docs disagree with it, it wins.

## What Kytelink is

An open-source link-in-bio platform. This repo is a ground-up rewrite (v2)
that replaced the original codebase at the cutover commit `faa5f4d`
("kytelink v2!", PR #25). The v1 code and the `rewrite/` design docs behind
the rewrite live only in git history now — `git show faa5f4d^:rewrite/README.md`
and `git show faa5f4d^:rewrite/00-context.md` are the best starting points
for *why* the rewrite exists and how it's organized.

## Repo layout

```
apps/web       Next (Pages Router) :3000 — editor + public profile pages
apps/landing   Next (Pages Router) :3001 — kytelink.com marketing zone
apps/admin     Next (Pages Router) :3002 — admin.kytelink.com
apps/api       Fastify + tRPC :3003 — /trpc/*, /auth/*, /t/* (beacons), /internal/*

packages/schemas    zod-only, dependency-free — single source of truth for domain types
packages/db         Prisma client — NEVER imported by apps/web (profile data flows through the API)
packages/clickhouse  analytics client — off gracefully when CLICKHOUSE_URL is unset
packages/trpc       the app tRPC router, shared by apps/api and app clients
packages/ui         ProfileView lives here ONCE — public profile, editor preview, landing demo all mount it
packages/cdn        owned static assets + getCdnUrl/getLqipUrl + the S3 sync script
packages/emails     react-email templates
packages/config      shared tsconfig/eslint/prettier/tailwind presets — every app/package consumes these

tools/seed     seeds the local Postgres (base fixtures always; agent accounts when AGENT_MODE=true)
deploy         Caddyfile for the self-hosting edge (COMPOSE_PROFILES=...,proxy) — terminates
               TLS and issues certs on demand for users' custom domains
```

Runtime topology, caching ownership, and the full environment contract are
documented in the `02-architecture.md` design doc
(`git show faa5f4d^:rewrite/02-architecture.md`) — read it before touching
boot/env code.

## Conventions (full list: `git show faa5f4d^:rewrite/23-conventions.md`)

- Lowercase kebab-case for every file and folder. No PascalCase filenames.
- Named exports over default exports, except Next `pages/*` files (Next
  requires the default export there).
- Strict TypeScript everywhere. `any`, `@ts-ignore`, and `console.log` are
  lint errors (enforced by `packages/config`'s eslint preset).
- No comments except genuinely non-obvious constraints — no section
  banners, no narration, no commented-out code, no TODO litter.
- `apps/web` never imports `packages/db` directly (enforced by

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aleemrehmtulla/kytelink](https://github.com/aleemrehmtulla/kytelink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
