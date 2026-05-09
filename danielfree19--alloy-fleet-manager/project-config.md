---
trigger: always_on
description: > **Purpose**: When the conversation context is close to its limit and we
---

# CLAUDE.md — Handoff & Context Recovery

> **Purpose**: When the conversation context is close to its limit and we
> need to start fresh (or hand off mid-task), read this file FIRST. It's
> the cheapest way back into the project's mental model — much cheaper than
> re-reading the whole repo or the prior transcript.
>
> Last updated: 2026-04-25 (Phase 2 SSO — OIDC delivered;
> open-source release plumbing landed: Apache-2.0 LICENSE,
> CONTRIBUTING/SECURITY/COC/MAINTAINERS, GitLab CI pipeline,
> GoReleaser configs for the Terraform provider + fleetctl, npm
> publish wiring for `@fleet-oss/sdk`, multi-arch Docker buildx,
> automated GitHub mirror push. See `docs/ci-cd.md` and
> `docs/release.md`.
> Security hardening pass — `@fastify/helmet` (strict CSP +
> `frame-ancestors 'none'` + HSTS), `@fastify/rate-limit` on
> `/auth/login` and `/auth/sso/start/:id`, per-account lockout
> (5 failures / 15 min, audited as `auth.login.locked` +
> admin-clear via `PATCH /users/:id { unlock: true }` →
> `user.unlock`), pino redaction of auth headers + body fields,
> `recordAuditEvent` deny-list sanitizer, OIDC SSRF guard
> (`auth/sso/url-guard.ts`: pre-flight + custom `lookup` into
> `openid-client` via `custom.setHttpOptionsDefaults`).
> First test harness — `vitest` in `apps/fleet-manager`
> (`npm run test -w apps/fleet-manager`).).

---

## 0. Hard rules (from the user, must follow)

These came from `<user_rules>` and apply to every change you make:

1. **Prefer Node.js + Go.** This is the polyglot baseline; don't introduce
   Python/Rust/etc. without explicit user approval.
2. **Document everything you do** and **document app functionality**. After
   any non-trivial change update the relevant `docs/*.md` (and this file
   when scope shifts). README links are the table of contents.
3. **Don't rewrite code without checking that you don't cancel old logic.**
   Old code paths are often deliberately preserved (e.g. the legacy agent,
   the legacy `AGENT_BEARER_TOKEN` path). Read the surrounding code and
   any preceding docs before deleting or replacing.

Project-wide conventions worth keeping in mind:

- Code comments explain **why / trade-offs / non-obvious intent**, never
  narrate "what" the code does. (See past commits — that's the bar.)
- Imports of internal modules in `apps/fleet-manager` use the `.js`
  extension even from `.ts` files (NodeNext module resolution).
- Biome is the JS linter. Several pre-existing patterns produce warnings
  (`req.actor!` non-null assertions, `autoFocus`, static `id` props).
  Don't fight pre-existing patterns; follow whatever the file already
  uses. Only fix lints **you** introduced.

---

## 1. What this project is

Self-hosted, vendor-neutral replacement for Grafana Cloud Fleet Management.
Built around Grafana Alloy's native `remotecfg` block — Alloy itself is
the agent, no sidecar.

Top-level mental model:

```
[ Operator / CI / fleetctl / Terraform / SDK ]
              │ Bearer admin or fmt_… token
              ▼
       ┌──────────────────┐
       │  Fleet Manager   │  Fastify (Node.js, TS), Postgres
       │  /pipelines CRUD │  ────────► audit_events
       │  /users /roles   │
       │  /tokens /catalog│
       │  Connect RPC:    │
       │  collector.v1.   │  ◄── Alloy `remotecfg` polls
       │  CollectorService│      (legacy AGENT_BEARER_TOKEN OR
       └──────────────────┘       fmt_… token w/ collectors.poll)
```

The full architecture doc is `docs/architecture.md`. Read it once if you
haven't.

---

## 2. Repo layout (only the parts you'll edit)

```
apps/
  fleet-manager/        # Fastify API — primary (remotecfg) + legacy (REST)
    src/
      server.ts         # Top-level bootstrap; route registration order matters
      config.ts         # zod-parsed env config
      auth/             # Identity + RBAC: permissions.ts, middleware.ts,
                        # users.ts, sessions.ts, password.ts
      remotecfg/        # Connect RPC handler for Alloy polls
      routes/           # auth.ts users.ts tokens.ts pipelines.ts catalog.ts
                        # audit.ts collectors.ts (+ legacy: configs.ts etc.)
      services/         # audit.ts (recordAuditEvent), assemble.ts, etc.
      db/migrations/    # node-pg-migrate SQL — run via `npm run migrate`
  fleet-ui/             # Vite + React + TS + Tailwind admin SPA, served at /ui/
    src/
      api/              # types.ts (mirrors backend DTOs) + apiFetch helper
      pages/            # Top-level routed pages
      components/       # Shared bits: AuditEventList, PageHeader, …
      stores/           # Zustand stores (auth, toasts, cache)
      App.tsx           # Routing — public /login + RequireAuth-wrapped layout
  fleet-agent/          # LEGACY Node.js agent. Preserved, NOT the primary path.

packages/
  sdk/                  # @fleet-oss/sdk — TS client for Node/browser
    src/{types,client,errors,index}.ts
  shared/               # Shared TS types + zod schemas

terraform/
  provider-fleet/       # Native Go provider (HashiCorp Plugin Framework v1)
    internal/provider/
      provider.go       # Schema + Configure
      client.go         # stdlib HTTP wrapper + DTOs
      convert.go        # tfsdk ↔ DTO helpers
      pipeline_resource.go     pipeline_data_source.go

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielfree19/alloy-fleet-manager](https://github.com/danielfree19/alloy-fleet-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
