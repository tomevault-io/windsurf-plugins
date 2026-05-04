---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Start Commands

### Local Development
```bash
# Start local PostgreSQL (optional, for local dev)
docker-compose up -d

# Start Next.js dev server
npm run dev

# View Docker logs
docker-compose logs -f

# Stop services
docker-compose down
```

### Testing & Building
```bash
# Run tests
make test

# Type checking
make typecheck

# Linting
make lint

# Full build (CI-compatible)
make ci-build

# Setup local environment
make setup
```

### MANDATORY: Local Build Test Before Commit

**ALWAYS run the local build before creating any commit.** This catches TypeScript errors, import issues, and other problems that CI would catch later.

```bash
npm run build
```

This is a **blocking requirement** - do not commit code that fails the local build. The CI pipeline will reject PRs with build failures, so catching them early saves time and review cycles.

## Architecture Overview

### Tech Stack
- **Frontend**: Next.js 14 (TypeScript) - Full-stack with API routes
- **Database**: PostgreSQL via Supabase - user data, workspaces, API keys, signals, accounts
- **Authentication**: Supabase Auth (OAuth with PKCE)
- **Deployment**: Vercel (production, staging, and preview environments)
- **Background Jobs**: Vercel Cron

### Service Architecture
```
┌─────────────────────────────────────────────────────┐
│              Next.js Application                    │
│                  (Vercel)                           │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │              App Router                       │   │
│  │  - (auth)/ - Login, OAuth callback           │   │
│  │  - (dashboard)/ - Protected app pages        │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │              API Routes                       │   │
│  │  - /api/signals/* - Signal CRUD              │   │
│  │  - /api/integrations/* - External services   │   │
│  │  - /api/cron/* - Scheduled jobs              │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │              Business Logic                   │   │
│  │  - lib/heuristics/ - Signal detection        │   │
│  │  - lib/integrations/ - API clients           │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────┬───────────────────────────────────┘
                  │
┌─────────────────▼───────────────────────────────────┐
│              PostgreSQL (Supabase)                  │
│  - Row Level Security (RLS)                        │
│  - Supabase migrations                             │
│  - Multi-tenant with workspaces                    │
└─────────────────────────────────────────────────────┘
```

### Key Modules

#### Authentication (Supabase Auth)
- **OAuth Flow**: PKCE-based OAuth via Supabase
- **Session Management**: Supabase handles session cookies
- **Workspace Context**: Every user belongs to a workspace (multi-tenant)

#### Database (Supabase)
Tables managed via Supabase migrations:
- `workspaces` - Multi-tenant container (has slug, subscription status)
- `workspace_members` - Links users to workspaces with roles
- `api_keys` - API key auth for programmatic access
- `accounts`, `contacts`, `signals` - Core business entities
- `integrations` - PostHog, Stripe, Apollo settings

#### Integrations (`lib/integrations/`)
- **PostHog** (`posthog/`): Analytics events, persons, accounts
- **Stripe** (`stripe/`): Customer data, subscription status
- **Apollo** (`apollo/`): Company enrichment data
- **Attio** (`attio/`): CRM sync destination

#### Heuristics & Scoring (`lib/heuristics/`)
- **Signal Detection** (`signals/`): 20+ product usage signal detectors
- **Scoring Engine** (`engine.ts`): Health, expansion, churn risk scores (0-100)
- **Concrete Grades** (`concrete-grades.ts`): M100, M75, M50, M25, M10

### Project Structure
```
src/
├── app/
│   ├── (auth)/              # Auth pages (login, callback)
│   ├── (dashboard)/         # Protected app pages
│   │   ├── page.tsx         # Home/Setup
│   │   ├── signals/         # Signal list & detail
│   │   ├── playbooks/       # Automation rules
│   │   ├── backtest/        # Signal testing
│   │   ├── settings/        # Integrations & config
│   │   └── identities/      # User/account list
│   ├── api/                 # API route handlers
│   │   ├── cron/            # Vercel Cron endpoints
│   │   ├── signals/         # Signal CRUD
│   │   └── integrations/    # Integration management
│   └── layout.tsx           # Root layout
├── components/
│   ├── ui/                  # Shadcn/Radix component library
│   ├── signals/             # Signal-specific components
│   └── layout/              # Layout components
└── lib/
    ├── heuristics/          # Signal detection & scoring
    │   ├── signals/         # 20 signal detectors
    │   ├── engine.ts        # Scoring engine
    │   └── concrete-grades.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getbeton/inspector](https://github.com/getbeton/inspector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
