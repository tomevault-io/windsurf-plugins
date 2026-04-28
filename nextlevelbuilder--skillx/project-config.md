---
trigger: always_on
description: SkillX.sh — AI agent skills marketplace. Web marketplace + CLI + hybrid search engine. "The Only Skill That Your AI Agent Needs."
---

# CLAUDE.md

## Project

SkillX.sh — AI agent skills marketplace. Web marketplace + CLI + hybrid search engine. "The Only Skill That Your AI Agent Needs."

**Related documents:**
- Project initial brief: `./docs/project-initial-brief.md`
- Project overview: `./docs/project-overview-pdr.md`
- Project roadmap: `./docs/project-roadmap-pdr.md`
- Design guidelines: `./docs/design-guidelines.md`

## Tech Stack

Code standards: `./docs/code-standards.md`
Codebase summary: `./docs/codebase-summary.md`

- **Monorepo**: pnpm workspaces (`apps/web`, `packages/cli`)
- **Web**: React Router v7 + Vite + SSR on Cloudflare Workers
- **Database**: Cloudflare D1 (SQLite) + Drizzle ORM
- **Search**: FTS5 + Vectorize (768-dim, bge-base-en-v1.5) + RRF fusion
- **Auth**: Better Auth + GitHub OAuth (route: `api/auth/*`)
- **Cache**: Cloudflare KV (5min TTL search, 5min leaderboard)
- **Storage**: R2 (assets), Workers AI (embeddings)
- **CSS**: Tailwind v4 + `@theme` tokens (dark-only, mint accent `#00E5A0`)
- **CLI**: Commander.js + chalk + ora + conf (package name: `skillx-sh`, bin name: `skillx`)

## Directory Structure

```
apps/web/app/
├── routes/              # React Router pages + API handlers
│   ├── home.tsx         # Home page + leaderboard
│   ├── skill-detail.tsx # Skill page + ratings/reviews
│   ├── profile.tsx      # User favorites
│   ├── settings.tsx     # API key management
│   ├── auth-catchall.tsx# Better Auth handler
│   ├── api.search.ts    # Hybrid search API
│   ├── api.admin.seed.ts# Demo data seeding
│   └── api.skill-*.ts   # Rate, review, favorite APIs
├── components/          # UI components
│   ├── layout/          # Navbar, footer
│   ├── search-command-palette.tsx  # Cmd+K modal
│   └── leaderboard-table.tsx      # Sortable table
├── lib/
│   ├── db/              # schema.ts (Drizzle), queries
│   ├── auth/            # auth-server.ts, session-helpers.ts, api-key-utils.ts
│   ├── github/          # fetch-github-skill, scan-github-repo (Tree API)
│   ├── search/          # hybrid-search, vector-search, fts5-search, rrf-fusion, boost-scoring
│   ├── vectorize/       # embed-text, chunk-text, index-skill
│   └── cache/           # KV caching utilities
├── root.tsx             # App shell
└── app.css              # Tailwind + theme tokens

packages/cli/src/        # `skillx` npm package
├── commands/            # search, use, publish, report, config
├── lib/                 # api-client, config-store
└── index.ts             # Commander.js entry
```

## Commands

```bash
pnpm install              # Install all dependencies
pnpm dev                  # Start dev server (apps/web) → http://localhost:5173
pnpm build                # Build for production
pnpm typecheck            # TypeScript check
pnpm test                 # Run unit tests (vitest) once
pnpm test:watch           # Run unit tests in watch mode
pnpm seed                 # Seed DB (requires ADMIN_SECRET env var)

# Inside apps/web:
pnpm db:generate          # Generate Drizzle migrations
pnpm db:migrate           # Apply migrations locally
pnpm db:migrate:remote    # Apply migrations to remote D1
```

## Routes (apps/web/app/routes.ts)

| Pattern | File | Auth |
|---------|------|------|
| `/` | home.tsx | None |
| `/skills/:slug` | skill-detail.tsx | None |
| `/profile` | profile.tsx | Session |
| `/settings` | settings.tsx | Session |
| `/api/auth/*` | auth-catchall.tsx | Better Auth |
| `/api/search` | api.search.ts | None |
| `/api/skills/:slug` | api.skill-detail.ts | None |
| `/api/skills/:slug/references` | api.skill-references.ts | None (GET), Session/Key (POST) |
| `/api/skills/:slug/rate` | api.skill-rate.ts | Session/Key |
| `/api/skills/:slug/review` | api.skill-review.ts | Session/Key |
| `/api/skills/:slug/favorite` | api.skill-favorite.ts | Session/Key |
| `/api/skills/:slug/install` | api.skill-install.ts | Optional (API key or X-Device-Id) |
| `/api/skills/:slug/vote` | api.skill-vote.ts | Session |
| `/api/report` | api.usage-report.ts | Session/Key |
| `/api/user/api-keys` | api.user-api-keys.ts | Session |
| `/api/user/interactions` | api.user-interactions.ts | Session (graceful fallback) |
| `/api/skills/register` | api.skill-register.ts | Session/Key |
| `/api/admin/seed` | api.admin.seed.ts | Admin secret |

## Database Tables (Drizzle schema)

`skills`, `skill_references`, `ratings`, `reviews`, `favorites`, `votes`, `usageStats`, `apiKeys`, `installs`
Plus Better Auth tables: `user`, `session`, `account`, `verification`

## Key Patterns

System architecture: `./docs/system-architecture.md`
Search algorithm: `./docs/search-algorithm.md`

**Page routes**: Loader (server-side data) + Component (React) + Action (form handling)

**API routes**: Auth check → Validate input → DB operation → JSON response

**Auth**: `getSession(request, env)` for session, `requireAuth(request, env)` for redirect

**Search**: Query → Embed (Workers AI) → Vectorize + FTS5 parallel → RRF fusion → 8-signal Boost scoring (vector 50%, FTS5 21.5%, rating 3%, installs 2%, votes 0.7%, recency 1%, reviews 0.15%, favorites 0.15%) → Filter → Cache (KV)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nextlevelbuilder/skillx](https://github.com/nextlevelbuilder/skillx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
