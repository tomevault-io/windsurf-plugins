---
trigger: always_on
description: > Universal context file for AI coding assistants. See also: [CLAUDE.md](./CLAUDE.md) for Claude-specific context.
---

# Louez - AI Agent Context

> Universal context file for AI coding assistants. See also: [CLAUDE.md](./CLAUDE.md) for Claude-specific context.

## Overview

**Louez** is a multi-tenant, self-hosted equipment rental management platform. It provides rental businesses with inventory management, reservation handling, customer databases, and branded storefronts.

- **License**: MIT (open-source)
- **Monorepo**: Turborepo + pnpm workspaces
- **Language**: TypeScript (strict mode)
- **Framework**: Next.js 16 with App Router

## Architecture

### Multi-Tenant Model

Each `store` operates independently with its own:

- Products, categories, pricing tiers
- Customers and reservations
- Settings, branding, legal pages
- Team members (owner/member roles)

**Subdomain routing**:

- `app.domain.com` → Admin dashboard
- `{store-slug}.domain.com` → Public storefront

### Route Groups

| Group          | Path                            | Auth      | Purpose          |
| -------------- | ------------------------------- | --------- | ---------------- |
| `(auth)`       | `/login`, `/verify-request`     | Public    | Authentication   |
| `(dashboard)`  | `/dashboard/*`, `/onboarding/*` | Protected | Store management |
| `(storefront)` | `/{slug}/*`                     | Public    | Customer-facing  |

### Data Flow

**Server Actions (mutations)**:

```
Request → Middleware (subdomain detection) → Layout (auth check) → Page/Action
                                                     ↓
                                            getCurrentStore()
                                                     ↓
                                            Database (storeId filter)
```

**oRPC (queries/mutations)**:

```
Client Component → orpc.dashboard.*.queryOptions() → /api/rpc/[...path]
                                                           ↓
                                                    RPCHandler → Procedure middleware
                                                           ↓
                                                    getCurrentStore() / getCustomerSession()
                                                           ↓
                                                    Database (storeId filter)
```

## Tech Stack

| Layer         | Technology                           |
| ------------- | ------------------------------------ |
| Monorepo      | Turborepo + pnpm workspaces          |
| Runtime       | Node.js, Next.js 16, React 19        |
| Language      | TypeScript 5                         |
| Database      | MySQL 8, Drizzle ORM                 |
| Auth          | Better Auth (OAuth, Magic Link, OTP) |
| API           | oRPC (type-safe RPC)                 |
| Data Fetching | TanStack Query                       |
| Validation    | Zod                                  |
| UI            | Tailwind CSS 4, shadcn/ui, Base UI   |
| Forms         | TanStack Form                        |
| Payments      | Stripe Connect                       |
| Email         | React Email, Nodemailer              |
| PDF           | @react-pdf/renderer                  |
| i18n          | next-intl (fr, en)                   |

## Monorepo Architecture

This project uses **Turborepo** with **pnpm workspaces** for monorepo management.

### Workspaces

| Workspace    | Package Prefix | Purpose                      |
| ------------ | -------------- | ---------------------------- |
| `apps/*`     | `@louez/`      | Deployable applications      |
| `packages/*` | `@louez/`      | Shared libraries and configs |

### Task Pipeline

Turborepo manages task execution with automatic dependency resolution and caching.

**Cached tasks** (outputs stored for replay):

- `build` - Production builds (outputs: `dist/**`, `.next/**`)
- `lint` - ESLint checks
- `type-check` - TypeScript validation

**Non-cached tasks** (always run):

- `dev` - Development server (persistent)
- `db:*` - Database operations (side effects)
- `clean` - Cleanup

### Filtering

Run tasks for specific packages using `--filter`:

```bash
# Run dev for web app only
pnpm dev --filter=@louez/web

# Build a specific package
pnpm build --filter=@louez/db

# Type-check web and its dependencies
pnpm type-check --filter=@louez/web...

# Run for all packages except one
pnpm build --filter=!@louez/config
```

### Adding a New Package

1. Create directory in `packages/` or `apps/`
2. Add `package.json` with `"name": "@louez/package-name"`
3. Run `pnpm install` to link workspaces
4. Import in other packages: `import { x } from '@louez/package-name'`

## Directory Structure

```
louez/
├── apps/
│   └── web/                        # Next.js application
│       ├── app/                    # Next.js App Router
│       │   ├── (auth)/            # Login pages
│       │   ├── (dashboard)/       # Protected admin routes
│       │   ├── (storefront)/      # Public store routes [slug]/
│       │   └── api/               # API endpoints (including /api/rpc)
│       ├── components/
│       │   ├── ui/                # Base components (shadcn/ui)
│       │   ├── dashboard/         # Admin-specific
│       │   └── storefront/        # Customer-facing
│       ├── lib/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Synapsr/Louez](https://github.com/Synapsr/Louez) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
