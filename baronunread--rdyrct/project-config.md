---
trigger: always_on
description: Guide for AI coding agents (and humans) working in this repo. `CLAUDE.md` is a
---

# AGENTS.md

Guide for AI coding agents (and humans) working in this repo. `CLAUDE.md` is a
symlink to this file.

## What this is

**rdyrct**: an organization-based link shortener + QR generator that runs
entirely on **Cloudflare**: one Worker serves the API, the short-link redirects,
and the static SPA. Source of truth is **D1** (SQLite); the slug→destination hot
path is **KV**; custom domains use **Cloudflare for SaaS**. Product host:
`rdyrct.com`. Public repo: `github.com/baronunread/rdyrct`.

## Tooling: use bun

Always use **bun**, never npm/npx.

```sh
bun install
bun run dev                 # vite dev (Worker + SPA) on :5173
bunx emulate --service resend   # local Resend inbox on :4000 (read: curl :4000/emails -H 'authorization: Bearer test_token_admin')
bun run db:migrate:local    # apply migrations to local D1
bun run db:reset:local      # wipe local D1 + KV, re-apply migrations (start from scratch; restart dev after)
bun add <pkg>               # dependencies
bunx agent-browser          # real-browser checks: screenshots, clicking through pages. Use it for any visual verification; do not hand-roll headless Chrome
bun scripts/seed-local.ts   # seed local D1/KV with fake data (run dev server first)
```

**Local Cloudflare state**: while `bun run dev` runs, the Explorer API at
`http://localhost:5173/cdn-cgi/explorer/api` exposes the local KV, R2, D1,
Durable Objects, and Workflows. Fetch that URL for the OpenAPI schema, then use
it to list, query, and manage local resources (e.g. inspect D1 rows or KV keys
without wrangler CLI calls).

**Two TypeScript projects; run BOTH after changes:**

```sh
bun run check                          # app + shared (tsconfig.json → src/app, src/shared)
bunx tsc -p tsconfig.worker.json --noEmit   # worker (src/worker)
bun run test                           # unit tests (bun test, tests/)
bun run doctor                         # react-doctor audit (React health score; --verbose for details)
bun run fallow                         # fallow codebase intelligence audit
```

react-doctor runs in CI through `.github/workflows/react-doctor.yml` (advisory,
PRs + main). Run fallow locally when auditing codebase health; track its
findings in issues rather than blocking CI.

react-doctor also runs as a pre-commit hook on staged files (`--blocking warning`).
react-doctor skill lives in `.agents/skills/react-doctor` and `.claude/skills/react-doctor`.
fallow skill lives in `.claude/skills/fallow`.

Shell writes to repo files are sandboxed; edit through the editor tools, not
`sed`/`perl` (or run bash with the sandbox disabled for scripted edits).

## Architecture

- **Worker** (`src/worker/index.ts`): custom-domain redirect middleware →
  BetterAuth at `/api/auth/*` → Polar webhook `/api/webhooks/polar` → API router
  (`/api/*`, behind `withSession`) → root `/:slug` redirect → SPA asset fallback.
- **Routing has NO `/app` prefix.** `/` is the marketing landing. Public routes:
  `/login`, `/signup`, `/privacy`, `/terms`, `/reset-password`,
  `/invite/:token`. The app lives at root keywords: `/dashboard` (quick link
  creation, quick stats, recent activity), `/analytics` (the full stats page), `/links`,
  `/domains`, `/members`, `/billing`, `/settings`, `/admin`. There is **no org id
  in URLs**: the current org is a localStorage-backed store, `useCurrentOrg`
  (`src/app/lib/current-org.ts`). Those keywords are reserved from custom slugs
  via `RESERVED_SLUGS` in `src/worker/util.ts` (the Worker also guards `/:slug`).
- **Billing is per-user, not per-org.** `user.plan` (`free`|`hobby`|`pro`) +
  Polar customer/subscription ids live on the user; each Polar product maps to
  a plan via `POLAR_*_PRODUCT_ID`. An org's effective limits are **its
  owner's plan**: `orgPlan()` in `src/worker/plan.ts` resolves the owner. Only
  Pro raises the owned-org cap above 1. Caps: `PLAN_LIMITS` in
  `src/shared/types.ts` (`{ orgs, links, members, domains, qr,
analyticsDays }`). Slugs on the **shared** domain are always random (every
  plan); chosen slugs exist only on custom domains, so the shared namespace
  can't be squatted. New users get **no default org**: there is no onboarding
  route; org-scoped pages render `NoOrgState`
  (`src/app/components/no-org.tsx`) until they create one, and `/billing`
  works org-less, so landing paid CTAs (`/signup?next=/billing?plan=…`) can
  check out before the first org exists (`/onboarding` redirects to
  `/dashboard`).
- **Auth**: BetterAuth (email+password, `requireEmailVerification` via the
  `emailOTP` plugin, 6-digit code; password reset stays a link). PBKDF2/WebCrypto
  hashing (`src/worker/password.ts`). The account matching the `SUPERADMIN_EMAIL`
  secret is the platform admin; admin routes **404** (not 403) for everyone else.
  Platform admins can **ban** users (`user.banned`): banning wipes their sessions
  and `databaseHooks.session.create.before` (in `better-auth.ts`) refuses new
  ones, while their orgs/links keep working.
  Self-service account deletion is blocked while the user still owns an org.
- **KV keys**: `slug:{slug}` (shared host), `slug:{host}:{slug}` (custom domain),
  `domain:{host}`. D1 is authoritative; KV is the redirect hot path. Clicks are

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baronunread/rdyrct](https://github.com/baronunread/rdyrct) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
