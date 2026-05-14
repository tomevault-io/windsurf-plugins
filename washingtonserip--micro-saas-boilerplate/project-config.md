---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Micro SaaS boilerplate built with a monorepo architecture using Turborepo. The stack includes Next.js 16 with App Router, tRPC v11 for type-safe APIs, Drizzle ORM with PostgreSQL, Better Auth for authentication, Stripe for payments, and shadcn/ui components with TailwindCSS v4.

**Package Manager:** pnpm 9.0.0+ (required)
**Node Version:** 18+

## Monorepo Structure

```
apps/
  web/              # Next.js application (main app)
packages/
  api/              # tRPC API layer (@repo/api)
  db/               # Drizzle ORM + schemas (@repo/db)
  ui/               # shadcn/ui components (@repo/ui)
  eslint-config/    # Shared ESLint configs
  typescript-config/# Shared TypeScript configs
```

## Common Commands

### Development
```bash
# Start all apps in dev mode (Next.js on port 3000)
pnpm dev

# Start only the web app
pnpm --filter web dev

# Build all packages and apps
pnpm build

# Type checking across all packages
pnpm check-types

# Lint all packages
pnpm lint

# Format code with Prettier
pnpm format
```

### Database Commands

**Important:** Always prefix database commands with `DATABASE_URL` when running outside of a configured Next.js environment.

```bash
# Generate migration files from schema changes
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/micro_saas_boilerplate" pnpm --filter @repo/db db:generate

# Run migrations (production workflow - creates migrations/ folder)
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/micro_saas_boilerplate" pnpm --filter @repo/db db:migrate

# Push schema directly to DB (dev only, skip for production)
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/micro_saas_boilerplate" pnpm --filter @repo/db db:push

# Open Drizzle Studio GUI at localhost:4983
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/micro_saas_boilerplate" pnpm --filter @repo/db db:studio

# Drop migrations
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/micro_saas_boilerplate" pnpm --filter @repo/db db:drop
```

**Note:** The `postinstall` script in `packages/db/package.json` automatically runs migrations when `DATABASE_URL` is set (used in production deployments).

### Docker Database
```bash
# Start PostgreSQL (localhost:5432) + pgAdmin (localhost:5050)
docker compose up -d

# Stop services
docker compose down
```

pgAdmin credentials: `admin@admin.com` / `admin`

### Better Auth Commands
```bash
# Generate a new secret for BETTER_AUTH_SECRET
npx @better-auth/cli secret

# Or use the dedicated generate-secret command
npx @better-auth/cli generate-secret
```

### Stripe Development
```bash
# Install Stripe CLI (macOS)
brew install stripe/stripe-cli/stripe

# Login to Stripe
stripe login

# Forward webhooks to local dev server (keep running during development)
stripe listen --forward-to http://localhost:3000/api/auth/stripe/webhook
```

## Architecture Details

### Authentication Layer (Better Auth)

Better Auth is configured in `apps/web/lib/auth.ts` with:

- **Adapter**: Drizzle adapter connected to PostgreSQL
- **Email/Password Auth**: Enabled with 8-128 character passwords
- **Email Verification**: Sends verification emails via Resend on signup
- **Password Reset**: Email-based password reset flow
- **Session Management**: 7-day sessions, updates every 24 hours
- **Stripe Integration**: Better Auth Stripe plugin for subscription management

**Auth Schema**: `packages/db/src/schema/auth.ts`
- Tables: `user`, `session`, `account`, `verification`
- Auto-created by Better Auth adapter

**Client Usage**:
```typescript
import { authClient } from "@/lib/auth-client";

// Sign up
await authClient.signUp.email({ email, password, name });

// Sign in
await authClient.signIn.email({ email, password });

// Get session
const session = await authClient.getSession();
```

**Server Usage**:
```typescript
import { auth } from "@/lib/auth";

const session = await auth.api.getSession({ headers: await headers() });
```

### Stripe Integration

Stripe is integrated via the Better Auth Stripe plugin (`@better-auth/stripe`):

- **Configuration**: `apps/web/lib/auth.ts` (stripePlugin)
- **Schema**: `packages/db/src/schema/stripe.ts`
  - Tables: `stripe_customer`, `stripe_subscription`
  - Automatically managed by Better Auth
- **tRPC Router**: `packages/api/src/routers/stripe.ts`
  - Procedures: `getSubscriptionStatus`, `createCheckoutSession`, `createBillingPortalSession`, `hasActiveSubscription`
- **Client Helpers**: `apps/web/lib/stripe/`
  - `helpers.ts`: Utility functions for Stripe operations
  - `subscription-guards.tsx`: React components and hooks for feature gating

**Subscription Plans**:
- **Free**: Default tier, no payment required
- **Starter**: Monthly/yearly with 14-day free trial
- **Pro**: Monthly/yearly with 14-day free trial

**Usage Example**:
```typescript
import { useHasAccess, RequireSubscription } from "@/lib/stripe/subscription-guards";

// Guard component
<RequireSubscription requiredPlan="starter">
  <PremiumFeature />
</RequireSubscription>

// Hook for access control
const { hasAccess, plan } = useHasAccess("pro");
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [washingtonserip/micro-saas-boilerplate](https://github.com/washingtonserip/micro-saas-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
