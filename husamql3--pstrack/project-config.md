---
trigger: always_on
description: PStrack is a competitive programming accountability platform. Users join groups, solve one daily problem from the NeetCode 250 roadmap, and get auto-verified against LeetCode/Codeforces APIs. Points, streaks, and badges create the motivation loop. The tagline: **Show up. Solve. Repeat.**
---

# PStrack - App Context

## What It Is

PStrack is a competitive programming accountability platform. Users join groups, solve one daily problem from the NeetCode 250 roadmap, and get auto-verified against LeetCode/Codeforces APIs. Points, streaks, and badges create the motivation loop. The tagline: **Show up. Solve. Repeat.**

## Branching

See `docs/BRANCHING.md`. `stage` is the default working branch; `main` is the stable release branch. Start feature and fix branches from `stage`, and do not push or open pull requests unless explicitly asked.

## Core Loop

1. Midnight - Trigger.dev cron assigns the next NeetCode 250 problem to all active groups
2. User clicks "Mark as Solved"
3. `verify-submission` job polls LeetCode/Codeforces for an accepted submission matching the problem + timestamp
4. If found: +10 points, streak incremented, badges evaluated
5. If neither solved nor paused by end of day: `mark-missed` job fires → -3 points, streak breaks

## User Actions Per Day

| Action | Outcome                                            |
| ------ | -------------------------------------------------- |
| Solve  | +10 pts, +5 if first in group, streak continues    |
| Pause  | Streak preserved, −5 pts, consumes 1 pause |
| Miss   | -3 pts, streak resets                              |

## Points & Gamification

- **+10** daily solve, **+5** first in group, **-3** miss
- Streak multipliers: 7d → 1.2x, 30d → 1.5x (bonus logged separately)
- Badges: Streak 7/30/100, First Solver 10/50, Consistent 30
- Leaderboard: group (all tiers) and global top 100 (Pro only), filterable by week/month/all-time

## Tiers

|                    | Free                    | Pro ($5 one-time)             |
| ------------------ | ----------------------- | ----------------------------- |
| Groups             | Join 1 (max 30 members) | Join up to 5 (max 50 members) |
| Private groups     | No                      | Yes (creator perk)            |
| Pauses/month       | 2                       | 4                             |
| Global leaderboard | No                      | Yes                           |
| Profile badge      | No                      | Yes                           |

Pro is a **lifetime one-time purchase** via Polar ($5). No subscriptions.

## Commands

Runtime is **Bun 1.3.14**. `.bun-version`, CI, `package.json`, and Docker pin the same release.

### Develop

| Command | What it does |
|---|---|
| `bun run dev` | Starts the portless proxy + Vite dev server. App is served at **https://pstrack.localhost** (not `127.0.0.1:PORT`). |
| `bun run dev:app` | Vite dev server only, bound to `127.0.0.1:${PORT:-3001}`. Use when you need a direct port. |
| `bun run dev:trigger` | Trigger.dev local dev runner (background jobs). |
| `bun run dev:email` | React Email preview server on port 3001 (`src/emails/`). |
| `bun run preview` | Vite preview of the production build. |

### Build & verify

| Command | What it does |
|---|---|
| `bun run build` | `prisma generate && vite build`. The TanStack Start build. |
| `bun run typecheck` | `prisma generate && tsc --noEmit`. Use this — never bare `tsc`, the generated Prisma client is required for types to resolve. |
| `bun run check` | Biome lint/format check (read-only). |
| `bun run format` | Biome write — applies fixes including unsafe ones. |
| `bun run knip` | Detects unused exports/files. |
| `bun run test` | Vitest, single run (no watch). |
| `bun run test -- src/path/to/file.test.ts` | Run one test file. Append `-t "name"` to filter by test name. |

### Database (Prisma → PostgreSQL)

| Command | What it does |
|---|---|
| `bun run db:generate` | Regenerate the Prisma client into `generated/prisma/`. Required after schema edits. |
| `bun run db:migrate` | `prisma migrate dev` — create + apply a dev migration. |
| `bun run db:migrate:deploy` | Apply pending migrations (CI / prod). |
| `bun run db:push` | Push schema without a migration (prototype use only). |
| `bun run db:seed` | Run the master seed (`prisma/seed.ts`). Also `db:seed-problems`, `db:seed-groups`, `db:seed-daily-problems` for targeted seeds. |
| `bun run db:reset` | `prisma migrate reset` — drops, re-migrates, re-seeds. Destructive; confirm before running. |
| `bun run db:studio` | Opens Prisma Studio against `.env`. |

### Env sync

| Command | What it does |
|---|---|
| `bun run env:sync:prod` | Sync `.env.prod` to the Coolify production app. |
| `bun run env:sync:stage:dry-run` | Preview the allowlisted `.env.stage` → Vercel staging sync. |
| `bun run env:sync:stage` | Sync the allowlisted `.env.stage` to Vercel staging. |
| `bun run env:sync:trigger` | Sync env vars to Trigger.dev. |
| `bun run env:sync:gh` | Sync env vars to GitHub Actions secrets. |

### Trigger.dev deploys

`bun run deploy:trigger-prod` deploys background jobs to Trigger.dev production. `bun run dev:trigger` is the local equivalent.

## Groups

- **Public** - request to join, admin approves/rejects within 1 day (auto-expires)
- **Private** - invite link only (7/30/90 day or permanent expiry), instant join

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [husamql3/pstrack](https://github.com/husamql3/pstrack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
