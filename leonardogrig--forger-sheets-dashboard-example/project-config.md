---
trigger: always_on
description: This is a Next.js 15.3.3 application using App Router with authentication (NextAuth), Stripe payments, and Prisma ORM. The project structure follows modern Next.js best practices with server components and TypeScript.
---

# Cursor Rules for Forger Starter Kit - Next.js 15 + Stripe

## Project Overview

This is a Next.js 15.3.3 application using App Router with authentication (NextAuth), Stripe payments, and Prisma ORM. The project structure follows modern Next.js best practices with server components and TypeScript.

## Architecture & Tech Stack

### Core Dependencies

- **Next.js**: 15.3.3 (App Router)
- **React**: 19.0.0
- **TypeScript**: 5.x
- **Authentication**: NextAuth 4.24.11 with Google Provider
- **Database**: PostgreSQL with Prisma 6.9.0 ORM
- **Payments**: Stripe 18.2.1
- **Styling**: Tailwind CSS 4.x with custom CSS variables
- **UI Components**: Radix UI primitives with shadcn/ui patterns
- **Package Manager**: Yarn

### File Structure

```
app/                     # Next.js App Router directory
  ├── api/              # API routes
  │   ├── auth/[...nextauth]/route.ts  # NextAuth configuration
  │   ├── stripe/
  │   │   ├── checkout/route.ts        # Stripe checkout sessions
  │   │   └── webhook/route.ts         # Stripe webhook handler
  │   └── admin/
  │       └── update-user/route.ts     # Admin user management
  ├── auth/signin/      # Authentication pages
  ├── dashboard/        # User dashboard
  ├── admin/           # Admin panel
  ├── pricing/         # Pricing page
  ├── premium-content/ # Protected premium content
  ├── free-content/    # Free content for authenticated users
  ├── banned/          # Banned user page
  └── layout.tsx       # Root layout with providers

components/             # Reusable components
  ├── ui/              # shadcn/ui components (button, card, etc.)
  ├── auth/            # Authentication components
  ├── dashboard/       # Dashboard-specific components
  ├── admin/           # Admin panel components
  ├── pricing/         # Pricing components
  └── providers/       # Context providers

lib/                   # Utility libraries
  ├── auth.ts          # NextAuth configuration & session management
  ├── stripe.ts        # Stripe client configuration
  ├── stripe-admin.ts  # Stripe admin utilities & webhook handlers
  ├── subscription.ts  # Subscription management utilities
  ├── prisma.ts        # Prisma client
  └── utils.ts         # General utilities

prisma/               # Database configuration
  ├── schema.prisma   # Database schema
  ├── migrations/     # Database migrations
  └── seed.ts         # Database seeding scripts

types/
  └── next-auth.d.ts  # NextAuth type extensions
```

## Development Rules & Guidelines

### NEVER USE

- `middleware.ts` - Authentication and routing should be handled in components/pages
- `"use client;"` in `layout.tsx` or `page.tsx` files - Only use in actual components
- `yarn build` unless specifically requested - Assume `yarn dev` is running

### Always Use

- **Package Manager**: `yarn` for all package operations
- **Server Components**: Default to Server Components for data fetching
- **App Router**: Use Next.js 15 App Router patterns
- **TypeScript**: Strict typing for all code

### Route Parameters

When a route uses params, always follow this pattern:

```typescript
// CORRECT
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  // use id here
}

// WRONG
export default async function Page({ params }: { params: { id: string } }) {
  // direct usage without awaiting
}
```

### Styling Guidelines

- **Primary Styling**: All styles in `globals.css` using Tailwind CSS
- **Design System**: Uses custom CSS variables for theming (light/dark mode support)
- **UI Components**: Use existing shadcn/ui components from `components/ui/`:
  - `badge`, `button`, `calendar`, `card`, `dialog`, `form`, `input`, `label`, `popover`, `select`, `sonner`
- **Loading States**: Use Skeleton components from shadcn (check if specific skeleton exists first)
- **Icons**: Use Lucide React For icons.

### Authentication & Authorization

- **Auth Provider**: NextAuth with Google OAuth
- **Session Management**: `getSession()` from `lib/auth.ts`
- **User Roles**: `USER`, `PREMIUM`, `ADMIN`, `BANNED` (defined in Prisma schema)
- **Route Protection**: Check authentication in page components, redirect to `/auth/signin`
- **Admin Access**: Use `isUserAdmin(userId)` from `lib/subscription.ts`

### Database & Prisma

- **ORM**: Prisma with PostgreSQL
- **Client Location**: Generated to `app/generated/prisma`
- **Key Models**: `User`, `StripeProduct`, `StripePrice`, `UserSubscription`, `StripeCustomer`
- **Subscription Management**: Use utilities from `lib/subscription.ts`

### Stripe Integration

- **Configuration**: `lib/stripe.ts` for client, `lib/stripe-admin.ts` for server operations
- **Checkout Flow**: POST to `/api/stripe/checkout` with `priceId`
- **Webhooks**: Handled in `/api/stripe/webhooks/route.ts`
- **Products/Prices**: Synced via webhooks to database
- **Customer Management**: Auto-create/retrieve customers in checkout flow

### API Routes Structure

```
/api/auth/[...nextauth]     # NextAuth handler
/api/stripe/checkout        # Create checkout sessions
/api/stripe/webhooks         # Process Stripe webhooks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leonardogrig/forger-sheets-dashboard-example](https://github.com/leonardogrig/forger-sheets-dashboard-example) — distributed by [TomeVault](https://tomevault.io/claim/leonardogrig).
<!-- tomevault:4.0:windsurf_rules:2026-04-18 -->
