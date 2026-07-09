---
trigger: always_on
description: - **`ARCHITECTURE.md`** — High-level overview with links to all feature docs
---

# CLAUDE.md

## Documentation Structure

- **`ARCHITECTURE.md`** — High-level overview with links to all feature docs
- **`FEATUREDOCS/`** — Individual markdown files for each feature/system
- **`PROMPT.md`** — Full product spec
- **`docs/ROADMAP.md`** — Prioritised roadmap: phases, sequencing, effort
- **`docs/designs/`** — Per-initiative design docs (one per major feature/program)

**When making changes**: Read the relevant `FEATUREDOCS/` file(s) for the feature you're touching. Update them after. Don't read everything — just what's relevant. The [Integration Checklist](./FEATUREDOCS/29-integration-checklist.md) tells you what to wire up for new features.

## Branching

All new features and non-trivial changes must go on a dedicated branch. Never commit feature work directly to `main`.

## Commits

Make atomic commits — one logical change per commit. The more commits the merrier. Prefer many small, focused commits over fewer large ones. Each commit should be independently understandable and revertable.

## Feature Documentation

Every feature change **must** update the relevant `FEATUREDOCS/` file. If the feature doesn't have one yet, create a new numbered markdown file (e.g. `FEATUREDOCS/30-my-feature.md`) and add it to the table in `ARCHITECTURE.md`.

## Commands

```bash
npm run dev          # Dev server (Turbopack, Next.js 16 default)
npm run build        # Production build + type check
npm start            # Start production server
npm run lint         # ESLint
npm test             # Run all unit tests
npm run test:watch   # Run tests in watch mode
npm run test:coverage # Run tests with coverage report
npx prisma generate  # Regenerate Prisma client (after schema changes)
npx prisma migrate dev --name <name>  # Create + apply migration
```

### Worktree Setup

Git worktrees don't share `node_modules/` or `.env` with the main repo. Run this to bootstrap a new worktree:

```bash
# Copy .env from main repo (adjust path if needed)
cp /Users/jayden/code/ttp-assetmanagement/.env .

# Install dependencies
npm install --legacy-peer-deps

# Generate Prisma client
npx prisma generate
```

After this, `npm run dev`, `npm test`, and `npm run build` will all work.

### Convex Dev in Worktrees

**Always use `pnpm exec convex` — never `npx convex`.** `npx convex` runs a global
CLI copy that can't resolve `convex/server` from local `node_modules`, causing an
esbuild failure. `pnpm exec convex` uses the locally installed version.

**When Claude Code edits `convex/*.ts` files**, push the changes immediately after:
```bash
pnpm exec convex dev --once
```
This is a one-shot push to the shared dev deployment — no watcher, no URL rewriting.
Run it automatically after any Convex function change. `CONVEX_DEPLOY_KEY` must be
in `.env`.

**When a human dev wants a live watcher**, use a named preview deployment to avoid
conflicting with other worktrees or the shared dev deployment:

```bash
# Start Convex watcher for this branch (creates/reuses a preview deployment)
pnpm exec convex dev --preview-run $(git rev-parse --abbrev-ref HEAD)
```

This writes the preview deployment URL to `.env.local` as `NEXT_PUBLIC_CONVEX_URL`,
which the dev server picks up automatically. Run it in a separate terminal alongside
`npm run dev`. The preview deployment name must not contain `/` — for worktree branches
like `feature/my-thing`, the branch name works fine as-is (Convex URL-encodes it).

`CONVEX_DEPLOY_KEY` must be set in `.env` or `.env.local` pointing to your Convex
Cloud project deploy key.

### DB Setup (first time)
```bash
# Ensure DATABASE_URL is set in .env, then:
npx prisma migrate dev   # Apply all migrations + generate client
```

## Environment Variables

**Required:**
- `DATABASE_URL` — PostgreSQL connection string
- `BETTER_AUTH_SECRET` — Auth signing secret
- `BETTER_AUTH_URL` — App base URL (used for auth callbacks)
- `NEXT_PUBLIC_APP_URL` — Public app URL (e.g. `http://localhost:3000`)

**Email (Resend):**
- `RESEND_API_KEY` — Resend API key (dev logs to console if unset)
- `EMAIL_FROM` — Sender address (default: `GearFlow <noreply@gearflow.app>`)

**File Storage (S3):**
- `S3_REGION`, `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`
- `S3_BUCKET` (default: `gearflow-uploads`)
- `S3_ENDPOINT` — Custom endpoint (optional, for S3-compatible providers)

**Google Maps:**
- `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY` — API key with Maps JavaScript API + Places API (New) enabled

**Other:**
- `PASSKEY_RP_ID` — WebAuthn relying party ID (default: `localhost`)
- `PLATFORM_NAME` — Display name (default: `GearFlow`)
- `ADMIN_REGISTRATION_TOKEN` — Secret token for `/register/admin?token=...`

**DB connection hardening (optional, safe defaults):** layered onto the runtime
`DATABASE_URL` in `src/lib/db-url.ts` (NOT onto `prisma migrate`, so backfills
aren't killed). Anything you put in the URL itself wins.
- `DB_STATEMENT_TIMEOUT_MS` — per-query server-side cap (default `30000`). The key
  stability guard: stops one slow query from holding a pooled connection and
  stalling the whole app. `0` disables (not advised).
- `DB_POOL_TIMEOUT_S` — wait for a free pooled connection before erroring (default `10`).
- `DB_CONNECTION_LIMIT` — max pooled connections (default: Prisma's `cpus * 2 + 1`).

## Critical Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TwoToned/gearflow](https://github.com/TwoToned/gearflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
