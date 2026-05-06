---
trigger: always_on
description: TuvixRSS is a modern RSS reader with AI features, built on Cloudflare Workers.
---

# TuvixRSS - Claude Code Guidelines

TuvixRSS is a modern RSS reader with AI features, built on Cloudflare Workers.

## Tech Stack

- **API**: Hono (Cloudflare Workers), tRPC, Drizzle ORM, Cloudflare D1
- **Frontend**: React, TanStack Router, TanStack Query, Tailwind CSS
- **Auth**: Better Auth (email/password)
- **Observability**: Sentry (errors, performance, metrics)
- **Email**: Resend
- **Monorepo**: pnpm workspaces (`packages/api`, `packages/app`, `packages/tricorder`)

## Project Structure

```
packages/
  api/          # Cloudflare Workers API (Hono + tRPC)
    src/
      routers/  # tRPC route handlers
      services/ # Business logic (RSS fetching, email, etc.)
      auth/     # Better Auth configuration
      db/       # Drizzle schema and migrations
  app/          # React frontend (Vite + TanStack)
  tricorder/    # RSS/Atom feed discovery library
```

## Critical Rules

### Production Database Operations

**⛔ NEVER run production database migrations or modifications without explicit user permission.**

This includes but is not limited to:

- `wrangler d1 execute <db> --remote`
- Any SQL migrations against production databases
- Schema alterations on live systems
- Data modifications in production

**Required Process:**

1. Generate migrations locally
2. Show the user what will change
3. Explain impact and safety
4. **ASK FOR PERMISSION**
5. Only after explicit approval, proceed

**Rationale:** Production database operations are irreversible and can cause data loss, service disruption, or schema conflicts. Always give the user control over these decisions.

**Exception:** Local/dev database operations (`--local`, `db:migrate:local`) are safe to run without asking.

### Production Deployments

**⛔ NEVER deploy to production. Only local development is allowed.**

Deployment is explicitly forbidden and handled by CI/CD pipelines.

### Staging Deployments

**✅ Staging deployments are allowed via manual workflow dispatch.**

Staging provides a production-like environment for testing PRs before they're merged to main.

**How to Deploy to Staging:**

1. Go to **Actions** → **Deploy to Staging** → **Run workflow**
2. Select the branch/PR to deploy (default: `main`)
3. Choose whether to seed test data (optional)
4. Click **Run workflow**

**What Happens:**

- API and App are deployed to staging environment
- **Staging database is wiped clean** (all data deleted)
- Fresh migrations are applied from scratch
- Optional test data seeding (if selected)

**Key Points:**

- Staging uses separate infrastructure (D1 database, Worker, Pages project)
- Database starts fresh on every deployment (no migration conflicts)
- Last deployment wins (concurrent deployments are cancelled)
- Perfect for testing PRs in a production-like environment

**Staging Secrets Required:**

```
STAGING_D1_DATABASE_ID              # Separate D1 database for staging
STAGING_VITE_API_URL               # Staging API URL
STAGING_CLOUDFLARE_PAGES_PROJECT_NAME  # Staging Pages project name
```

**When to Use Staging:**

- Test PRs before merging to main
- Verify database migrations work correctly
- Integration testing with production-like infrastructure
- Demo features to stakeholders

**When NOT to Use Staging:**

- Local development (use `pnpm dev` instead)
- Quick iteration (too slow compared to local)
- Testing that requires preserving data (staging wipes on each deploy)

## Common Workflows

### Database Changes

1. Modify schema in `packages/api/src/db/schema.ts`
2. Generate migration: `pnpm db:generate`
3. Review generated SQL in `packages/api/drizzle/`
4. Apply locally: `pnpm db:migrate:local`

### Running Tests

- API: `pnpm --filter @tuvixrss/api test`
- App: `pnpm --filter @tuvixrss/app test`
- All: `pnpm test`

### Type Checking & Linting

- `pnpm type-check` - Check all packages
- `pnpm lint` - Lint all packages
- `pnpm format` - Format with Prettier

## Key Architecture Decisions

- **Fire-and-forget emails**: Email sending doesn't block API responses; uses Sentry spans for tracking
- **Admin dashboard**: User management at `packages/api/src/routers/admin.ts`
- **Security audit logging**: All auth events logged to `security_audit_log` table
- **Rate limiting**: Cloudflare Workers rate limit API per plan tier

## AI Features Configuration

TuvixRSS includes optional AI-powered features using OpenAI and the Vercel AI SDK.

### Features

- **AI Category Suggestions**: Automatically suggests feed categories based on feed metadata and recent articles
- **Model**: GPT-4o-mini (via `@ai-sdk/openai`)
- **Location**: `packages/api/src/services/ai-category-suggester.ts`

### Feature Access Control

AI features are **triple-gated** for security and cost control:

1. **Global Setting**: `aiEnabled` flag in `global_settings` table (admin-controlled via admin dashboard)
2. **User Plan**: Only Pro or Enterprise plan users have access
3. **Environment**: `OPENAI_API_KEY` must be configured

Access check: `packages/api/src/services/limits.ts:checkAiFeatureAccess()`

### Configuration

**Local Development (Docker/Node.js):**

```env
# Add to .env
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxxx
```

**Cloudflare Workers (Production/Staging):**

```bash
# Use wrangler CLI to set secret
cd packages/api

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TechSquidTV/Tuvix-RSS](https://github.com/TechSquidTV/Tuvix-RSS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
