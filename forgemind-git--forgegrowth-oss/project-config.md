---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) and other coding agents working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) and other coding agents working in this repository.
Human contributors should read it too — it is the set of invariants that a plausible-looking change
can quietly break.

Start with [`README.md`](./README.md) for what the product does. This file is only about *how to
change it safely*.

---

## What this is

A self-hosted WhatsApp-native growth stack: Meta ad → click-to-WhatsApp conversation → lead →
funnel stage → payment, in one Express + React app over one PostgreSQL database.

Single-tenant by design. There is no workspace or organisation concept anywhere — every config
table is a singleton `id = 1` row. Do not add a `workspace_id` that would only ever hold one value.

## Stack and conventions

| | |
|---|---|
| Backend | Node.js 20, Express 4, **raw SQL via `pg`** — no ORM, no query builder |
| Frontend | React 18 + Vite, **inline styles only** — no Tailwind, no CSS modules |
| State | `useState` / `useEffect`. No Redux, no Zustand |
| Icons | `lucide-react` |
| Fonts | DM Sans for text, DM Mono for numbers |
| Types | Plain JS throughout. No TypeScript |

- **API responses are camelCase**, produced by aliasing in SQL (`col AS "colName"`), not by mapping
  in JS.
- **Errors are `{ error: "human readable message" }`** — never a stack trace, never a bare code.
- **Every frontend call goes through `frontend/src/api.js`.** Components do not call `fetch`.
- Match the file you are editing. Comment density, naming and structure vary between older and
  newer files; follow the local style rather than imposing one.

## Hard invariants

Breaking any of these produces a bug that looks like something else, which is why they are listed.

### The schema name is hardcoded

`coexistence` appears throughout the SQL. Isolation is **per-database**, not per-schema. Do not
parameterise it without changing every query.

### Queue names are hardcoded

Two deployments sharing one Redis server will consume each other's outbound WhatsApp sends unless
they use different Redis database indexes (`redis://redis:6379/0` vs `/1`).

### Stage changes are observed, not hooked

Eight code paths write `leads.stage`, several in raw SQL. Anything that must react to a stage change
walks the append-only `lead_events` log with a saved cursor (see `services/funnelTags.js`). This is
why a new write path is covered for free. **Add to that pattern; do not add a ninth hook** — the
ninth is the one someone forgets.

### Migrations run before the code that needs them

An extra column is ignored by the running backend, so a schema slightly ahead is harmless. Code
ahead of its schema throws on the first request touching the missing column. Migrations are plain
numbered SQL in `supabase/migrations/`, applied in filename order, and must be **idempotent**
(`CREATE TABLE IF NOT EXISTS`, guarded `ALTER`s) because re-running them is the normal upgrade path.

Several tables are additionally created by `ensure*Tables()` functions that run at backend startup,
so a fresh deploy self-heals. **If you change such a table in a migration, change the matching
`ensure*Tables()` too** — otherwise behaviour differs between a fresh install and an upgrade.

### Never introduce a sentinel-value equality check

An unconfigured env var is `''`. Comparing it directly against a record field means `'' === ''`
evaluates true for every record that also lacks the value, silently matching everything. See
`isBridgedNumber()` in `routes/webhook.js`: it returns false when unconfigured, deliberately. Do not
replace it with a bare equality.

### Webhook handling must never throw

Meta retries on non-2xx and times out at 20 seconds. Attribution, analytics and agent work are all
best-effort around message ingestion: catch, log, continue. Anything slow (agent runs, media
downloads) goes on a queue rather than running inline.

### Phone numbers are digits-only

Normalised on insert by `normalizePhone()` in `services/metaPayload.js`. `+91…` and `91…` must not
become two chat threads.

### Capabilities are global, not per-token

MCP capability toggles apply immediately to every already-connected client. There is no per-key
scoping, and code must not imply there is.

### The MCP proxy canonicalises before it gates

`fetch()` strips dot-segments, so gating on the raw path string once let `/marketing/../users`
through as `area_marketing` and then delivered it to `/api/users`. Canonicalise **first**, gate on
the result. Any change there must keep "the path we check" identical to "the path the server
receives".

### The public address has one definition

`util/publicUrl.js` decides what URL the outside world reaches this install on. The OAuth issuer,
the Streamable HTTP challenge, the MCP Tools panel and the plugin `.zip` all read it from there,
and the plugin download substitutes the whole **origin** — scheme included — not just the hostname.

Four hand-written copies of that logic disagreed on comma-joined `x-forwarded-host`, on whether
`https` was pinned, and on which env var to fall back to. What makes it worth an invariant is how
it fails: the server issues an OAuth token for the host the request arrived on, so a connector

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Forgemind-git/ForgeGrowth-OSS](https://github.com/Forgemind-git/ForgeGrowth-OSS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
