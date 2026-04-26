---
trigger: always_on
description: **PrepChef** is a two-sided commercial kitchen rental marketplace connecting culinary entrepreneurs with commercial kitchen space. Think "Airbnb for commercial kitchens." The repo lives at https://github.com/PetrefiedThunder/Prep and deploys to https://prep-ashen.vercel.app.
---

# CLAUDE.md — PrepChef Autonomous Development Guide

## Project Identity

**PrepChef** is a two-sided commercial kitchen rental marketplace connecting culinary entrepreneurs with commercial kitchen space. Think "Airbnb for commercial kitchens." The repo lives at https://github.com/PetrefiedThunder/Prep and deploys to https://prep-ashen.vercel.app.

**Status**: MVP Complete (Sprint 0–7). Now entering Phase 2 development.

**Founder context**: Solo founder, non-technical visionary. Code quality, security, and production-readiness matter more than speed. Every commit should make the product more trustworthy and closer to market.

## Tech Stack

| Layer | Technology | Notes |
|-------|-----------|-------|
| Framework | Next.js 14 (App Router) | Server and client components, server actions |
| Language | TypeScript | Strict mode, type safety everywhere |
| Styling | TailwindCSS | Utility-first, mobile-responsive |
| Database | Supabase (PostgreSQL) | Managed Postgres with Row Level Security |
| Auth | Supabase Auth | Email/password, JWT-based sessions |
| Payments | Stripe Connect | Two-sided marketplace payments, 10% platform fee |
| Hosting | Vercel | Serverless deployment |
| Logging | Custom structured logger | `lib/logger` — JSON format, auto-redacts secrets |

## Repository Structure

```
Prep/
├── app/                    # Next.js App Router pages and API routes
│   └── api/webhooks/stripe/ # Stripe webhook handler (idempotent booking creation)
├── components/             # React components
├── lib/                    # Shared utilities, Supabase clients, logger
├── middleware/              # Custom middleware modules
├── middleware.ts            # Root middleware (auth session refresh)
├── supabase/               # Database migrations and Supabase config
├── tests/                  # Test suite
├── docs/                   # Additional documentation
├── .github/                # CI/CD workflows
├── API.md                  # Server actions and API route documentation
├── SCHEMA.md               # Complete database schema with RLS policies
├── SECURITY.md             # Security posture documentation
├── DEPLOYMENT.md           # Production deployment guide
├── DEPLOYMENT_MEMO.md      # Deployment notes
└── INSTALL.md              # Local development setup
```

## Database Schema (6 tables)

1. **profiles** — Extends auth.users with email, full_name, phone. Auto-created via trigger on signup. RLS: users see/edit only their own profile.
2. **kitchens** — Listings with owner_id (FK → profiles), title, description, address, city, state, zip_code, price_per_hour, max_capacity, square_feet, is_active. RLS: owners manage their own, public can view active listings.
3. **kitchen_photos** — Photo URLs linked to kitchens. RLS: owners manage their own kitchen photos.
4. **stripe_accounts** — Stripe Connect account linking for kitchen owners. RLS: users see only their own.
5. **bookings** — Reservations with payment info, created idempotently via Stripe webhooks. RLS: owners see bookings for their kitchens, renters see their own bookings.
6. **payouts** — Owner payout tracking (90% revenue after 10% platform fee). RLS: owners see only their own payouts.

Full schema documentation: `SCHEMA.md`

## Architecture Pattern

```
Browser → Next.js Pages → Server Actions → Supabase Client (Browser)
                ↓
Vercel Edge: Middleware (Auth Refresh) → Server Actions → API Routes
                ↓                                         ↓
        Supabase Cloud                          Stripe API
     (PostgreSQL + RLS + Auth)       (Connect + Checkout + Webhooks)
```

Key patterns:
- **Server Actions** with `'use server'` for type-safe mutations
- **Middleware** refreshes auth sessions on every request
- **Stripe webhooks** create bookings idempotently (prevents duplicates)
- **RLS everywhere** — database-enforced access control, not application-level

## Phase 2 Roadmap (ACTIVE — Work on these)

These are the founder's defined next priorities. Work through them in order, one feature at a time. Each feature should be a complete, tested, production-ready implementation on its own branch before merging to main.

### Priority 1: Photo Upload to Supabase Storage
- Replace URL-based kitchen photos with actual file uploads
- Use Supabase Storage with signed URLs
- Add image optimization/resizing
- Update kitchen creation and edit flows
- Add drag-and-drop upload UI

### Priority 2: Calendar Availability Management
- Kitchen owners define available time slots
- Prevent double-booking at the database level
- Calendar UI for viewing and managing availability
- Integrate availability checking into the booking flow

### Priority 3: Email Notifications
- Booking confirmation emails (to both owner and renter)
- Booking reminder emails (24 hours before)
- New booking alerts for kitchen owners
- Use a service like Resend or Supabase Edge Functions

### Priority 4: Reviews and Ratings
- Renters can review kitchens after completed bookings
- Star ratings (1–5) with text reviews

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PetrefiedThunder) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
