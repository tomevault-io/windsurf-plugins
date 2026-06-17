---
trigger: always_on
description: Eggbook is a Progressive Web App marketplace where egg sellers create profile pages (`@username`), list eggs with tags/photos/pricing, and buyers can browse, request orders, message sellers, and pay via Stripe Connect.
---

# Eggbook - Copilot Instructions

## Project Overview

Eggbook is a Progressive Web App marketplace where egg sellers create profile pages (`@username`), list eggs with tags/photos/pricing, and buyers can browse, request orders, message sellers, and pay via Stripe Connect.

## Tech Stack

| Layer | Technology | Version |
|-------|------------|---------|
| Framework | Next.js (App Router) | 16.x |
| React | React | 19.x |
| Styling | Tailwind CSS | 4.x |
| Auth | Clerk | 7.x |
| Database | PostgreSQL + Prisma | 7.x |
| File Storage | Vercel Blob | - |
| Real-time | Pusher | - |
| Payments | Stripe Connect | - |
| Validation | Zod | 4.x |

## Critical: Version-Specific Guidance

### Next.js 16
This project uses Next.js 16 which has breaking changes from earlier versions. **Always consult `node_modules/next/dist/docs/` before writing Next.js code.** Do not rely on training data for Next.js APIs.

### Prisma 7
Prisma 7 requires driver adapters for direct database connections (no longer uses native engine binaries by default). The project uses `@prisma/adapter-pg` with a `pg` Pool. See `src/lib/db.ts` for the correct pattern.

### Tailwind CSS 4
Uses the new Tailwind v4 with `@tailwindcss/postcss`. Configuration may differ from v3.

### Zod 4
Schema validation uses Zod v4. Syntax may differ from v3.

## Code Conventions

### File Structure
```
src/
├── app/                    # Next.js App Router pages and API routes
│   ├── (auth)/            # Auth pages (grouped route)
│   ├── [username]/        # Dynamic seller profile pages
│   ├── admin/             # Admin dashboard
│   ├── api/               # API routes
│   ├── dashboard/         # Seller dashboard
│   └── components/        # Page-specific components (if any)
├── components/            # Shared UI components
│   ├── ui/               # Base UI primitives (button, card, input, etc.)
│   └── ...               # Feature components
├── lib/                   # Utilities and configurations
│   ├── auth.ts           # Clerk auth helpers
│   ├── db.ts             # Prisma client with pg adapter
│   ├── stripe.ts         # Stripe configuration
│   ├── pusher.ts         # Pusher configuration
│   └── utils.ts          # General utilities
prisma/
└── schema.prisma          # Database schema
```

### TypeScript
- Use strict TypeScript
- Prefer explicit types over `any`
- Use Zod for runtime validation at API boundaries

### React Components
- Use functional components with hooks
- Prefer Server Components by default, add `'use client'` only when needed
- Keep components small and focused

### API Routes
- All API routes are in `src/app/api/`
- Use route handlers (route.ts) with named exports (GET, POST, PUT, DELETE)
- Validate request bodies with Zod
- Return proper HTTP status codes

### Database
- Access Prisma via `import { db } from '@/lib/db'`
- Use transactions for multi-step operations
- Handle not-found cases explicitly

### Authentication
- Use Clerk for all auth
- Access current user via `import { auth, currentUser } from '@clerk/nextjs/server'`
- Protect API routes with auth checks
- User roles: BUYER, SELLER, ADMIN

### Feature & Subscription Checks
Use Clerk's `has()` function to check for features or subscription entitlements before allowing gated actions:

```typescript
import { auth } from '@clerk/nextjs/server'
import { canCreateListing } from '@/lib/subscription'

export async function POST(req: NextRequest) {
  const { userId, has } = await auth()
  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  // Clerk's has() is the authoritative subscription check
  const hasFeature = has({ feature: 'listing' })
  if (!hasFeature) {
    return NextResponse.json(
      { error: 'Subscription required', code: 'SUBSCRIPTION_REQUIRED' },
      { status: 403 }
    )
  }

  // DB check for listing limits only (not subscription status)
  const dbCheck = await canCreateListing(userId)
  if (!dbCheck.allowed) {
    return NextResponse.json(
      { error: dbCheck.reason, code: dbCheck.code },
      { status: 403 }
    )
  }
  // ... proceed with action
}
```

**Subscription model:**
- `has()` first for authoritative subscription check, then DB for limits only
- User model stores: `subscriptionId`, `subscriptionPlan`, `subscriptionStatus`, `subscriptionExpiresAt`, `listingLimit`
- SubscriptionStatus enum: NONE, ACTIVE, CANCELED, EXPIRED
- EggListing has `hiddenBySubscription` to track subscription-hidden listings

**Key features to check:**
- `listing` - Required for sellers to create egg listings (via `seller_plan`)
- Configure features in Clerk Dashboard under "Features" or via subscription plans

**Webhook events handled:**
- `subscription.created` → Activate subscription, restore hidden listings
- `subscription.updated` → Update status and plan details
- `subscription.deleted` → Expire subscription, hide all listings immediately

### Styling
- Use Tailwind CSS utility classes
- Use `clsx` and `tailwind-merge` (via `cn()` in utils) for conditional classes
- Keep consistent spacing and color usage

## Key Data Models


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cwdcwd/eggbook](https://github.com/cwdcwd/eggbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
