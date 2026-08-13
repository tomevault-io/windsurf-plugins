---
trigger: always_on
description: - **Framework:** Next.js 14 (App Router)
---

# Tandem — Agent Instructions

## Tech Stack
- **Framework:** Next.js 14 (App Router)
- **UI:** React 18, Tailwind CSS, shadcn/ui, Recharts (dashboard charts)
- **Language:** TypeScript (strict)
- **ORM:** Prisma 5
- **Database:** PostgreSQL
- **Auth:** NextAuth.js
- **Testing:** Jest

## Commands
- **Test:** `npm test`
- **Build:** `npm run build`
- **Dev server:** `npm run dev` (port 2000)
- **Lint:** `npm run lint`
- **DB migrate:** `npx prisma migrate dev`
- **DB generate:** `npx prisma generate`
- **DB push:** `npx prisma db push`

## Architecture
- **App Router:** All routes under `src/app/`
- **API routes:** `src/app/api/`
- **Pages:** `src/app/(dashboard)/`
- **Components:** `src/components/<feature>/`
- **Cascade engine:** `src/lib/cascade.ts` — core GTD dependency promotion logic
- **Service layer:** `src/lib/services/` — business logic (task-service, project-service, etc.)
- **Validation:** `src/lib/validations/` — Zod schemas for API input validation
- **Dashboard:** `src/components/dashboard/` — PM dashboard widgets (health, progress, velocity, blocked queue, stale projects, milestones, burn-down)
- **Dashboard API:** `GET /api/dashboard/stats` — single endpoint returning all dashboard widget data
- **Contexts:** `src/app/(dashboard)/contexts/` — GTD context management (@Home, @Computer, etc.) with auto-seeded defaults on first visit
- **Contexts API:** `GET/POST /api/contexts`, `PATCH/DELETE /api/contexts/[id]`
- **MCP stdio:** `src/mcp/server.ts` — standalone process for Claude Desktop/Claude Code
- **MCP HTTP:** `src/app/api/mcp/route.ts` — Streamable HTTP transport for claude.ai, ChatGPT, etc.
- **MCP shared:** `src/mcp/tools.ts`, `src/mcp/resources.ts`, `src/mcp/prisma-client.ts` (AsyncLocalStorage for per-request context)
- **Onboarding:** `src/components/onboarding/` — 6-step first-run wizard (Welcome, Brain Dump, Process One, Contexts, Areas, Done) with redirect from Do Now page
- **Onboarding API:** `GET /api/onboarding/status`, `POST /api/onboarding/complete`, `POST /api/onboarding/reset`
- **Notifications:** `src/components/notifications/` — bell icon, notification panel, push subscription hook
- **Notifications API:** `GET /api/notifications`, `PATCH /api/notifications/[id]`, `GET /api/notifications/unread-count`, `POST /api/notifications/mark-all-read`, `POST/DELETE /api/push-subscriptions`, `GET/PATCH /api/notification-preferences`, `POST /api/cron/notifications`
- **Push infra:** `src/lib/push.ts` (VAPID + sendPushToUser), `public/sw.js` (push + notificationclick handlers)

## Development & Deploy Workflow
1. **Feature branch** — create a branch off `main` (e.g. `feat/my-feature`)
2. **Develop locally** — make changes, test on `localhost:2000` (`npm run dev`)
3. **Commit & push** — commit to the feature branch and push to GitHub
4. **Deploy to beta** — pull the feature branch on the beta server, build, restart
   ```
   ssh tandem-vps "sudo -u tandembeta bash -c 'cd /opt/tandem-beta && git fetch && git checkout <branch> && npm run build'"
   ssh tandem-vps "sudo systemctl restart tandem-beta"
   ```
5. **Test on beta** — verify the feature works on the live beta server
6. **Merge to main** — once verified, merge the feature branch into `main` and push

**Important:** Never commit directly to `main`. Always use a feature branch.

## Server Infrastructure

All three instances run on the same bare-metal server (`tandem-vps` SSH alias) with separate Linux users, databases, and systemd services.

| | Beta | Alpha | Production |
|---|---|---|---|
| **URL** | beta.tandemgtd.com | alpha.tandemgtd.com | tandemgtd.com |
| **Branch** | `main` | `main` | `release/1.8` |
| **Linux User** | `tandembeta` | `tandemAlpha` | `tandemprod` |
| **App Path** | `/opt/tandem-beta` | `/opt/tandem-alpha` | `/opt/tandem-prod` |
| **Service** | `tandem-beta` | `tandem-alpha` | `tandem-prod` |
| **Port** | 2000 | 2100 | 2200 |

### Deploy beta + alpha (from main)
```bash
# Beta
ssh tandem-vps "sudo -u tandembeta bash -c 'cd /opt/tandem-beta && git pull origin main && npm run build'"
# Alpha
ssh tandem-vps "sudo -u tandemAlpha bash -c 'cd /opt/tandem-alpha && git pull origin main && npm run build'"
# Restart all
ssh tandem-vps "sudo systemctl restart tandem-beta tandem-alpha"
```

### Deploy production (from release/1.8)
```bash
ssh tandem-vps "sudo -u tandemprod bash -c 'cd /opt/tandem-prod && git pull origin release/1.8 && npm run build'"
ssh tandem-vps "sudo systemctl restart tandem-prod"
```

### Seed help articles (after adding/updating docs/help/ files)
```bash
ssh tandem-vps "sudo -u tandembeta bash -c 'cd /opt/tandem-beta && npx tsx prisma/seed-help.ts'"
ssh tandem-vps "sudo -u tandemAlpha bash -c 'cd /opt/tandem-alpha && npx tsx prisma/seed-help.ts'"
ssh tandem-vps "sudo -u tandemprod bash -c 'cd /opt/tandem-prod && npx tsx prisma/seed-help.ts'"
```

Full ops guide: `docs/ops/ALPHA_ENVIRONMENT.md`

## Important Conventions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [courtemancheatelier/tandem-gtd](https://github.com/courtemancheatelier/tandem-gtd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
