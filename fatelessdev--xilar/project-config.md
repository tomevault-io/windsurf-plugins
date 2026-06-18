---
trigger: always_on
description: > **Design Philosophy & Visual Styling:**
---

# XILAR E-Commerce — Copilot Instructions

> [!IMPORTANT]
> **Design Philosophy & Visual Styling:**
> For all design context, colors, typography, motion rules, animations, interactive mechanics, performance safeguards, and accessibility guidance, you **must** refer to [.impeccable.md](file:///c:/Users/fateless/Documents/Projects/xilar/.impeccable.md). Keep this instructions file focused on backend/frontend engineering architecture and implementation rules.

CHECK IF YOU HAVE SKILLS FOR THE TECHNOLOGY YOU'RE TRYING TO IMPLEMENT. IF PRESENT, REFER TO THE SKILLS FIRST AND THEN PROCEED. USE THE ASK QUESTIONS TOOL AT THE SLIGHTEST BIT OF CONFUSION

No bandaid fixes. Fix root causes.
No backward-compatibility shims (early-stage product).
For each task map out the full scope where the changes are needed like db, api, frontend etc. and then implement all at once.

## Tech Stack

- **Framework:** Next.js 16 (App Router), React 19, TypeScript 5
- **Database:** Neon PostgreSQL (serverless) via Drizzle ORM (`lib/db/schema.ts`, `lib/db/index.ts`)
- **Auth:** Better Auth with admin plugin (`lib/auth.ts`, `lib/auth-server.ts`, `lib/auth-client.ts`)
- **Payments:** Razorpay (order creation + signature verification) — `lib/razorpay.ts`, `app/api/razorpay/`
- **AI:** OpenRouter via Vercel AI SDK — `lib/openrouter.ts`, `app/api/bargain/route.ts`
- **Images:** Cloudinary CDN with auto-format/quality — `lib/cloudinary.ts`
- **Styling:** Tailwind CSS 4 + `cn()` utility from `lib/utils.ts`, Lucide icons, Framer Motion

## Architecture

```
app/              → Pages & API routes (App Router)
  admin/          → Admin dashboard (requires admin role)
  api/            → Route handlers (auth, products, orders, bargain, razorpay, coupons, upload)
  shop/           → Store pages with gender/category filtering
  product/[id]/   → Product detail (dynamic route)
components/
  features/       → Domain components (bargain-ai, cart-drawer, checkout-bargain, product-client, shop-client)
  layout/         → Navbar, sidebar
  ui/             → Primitives (Button, Card) — shadcn-style with variant props, NOT shadcn/ui
lib/
  actions/        → Server actions (admin.ts, orders.ts, bargain.ts) — all DB mutations go here
  checkout/       → Server-owned checkout quote, pricing, and validation module
  coupon-validation.ts → Public coupon validation and discount math; do not import public coupon rules from admin actions
  bargain/        → Bargain prompt, display labels, pure rules, and DB eligibility context
  db/             → Drizzle schema & connection
  *-context.tsx   → Client providers (cart, wishlist, theme) using localStorage
types/index.ts    → Shared TypeScript interfaces
drizzle/          → Migration SQL files
```

## Key Patterns

### Data Mutations & Server Actions
All database writes use server actions in `lib/actions/`. API route handlers call these actions — never write to DB directly from routes. Admin actions enforce `requireAdmin()` from `lib/auth-server.ts`.

### Auth Guards
- `getServerSession()` — get current session (pages/routes)
- `requireAuth()` / `requireAdmin()` — throw if unauthorized (server actions)
- Client: `useSession()` hook from `lib/auth-client.ts`

### Client State (Cart, Wishlist, Theme)
Stored in `localStorage` (keys: `xilar-cart`, `xilar-wishlist`, `xilar-theme`). Provided via React Context from `lib/*-context.tsx`. No server-side cart — entirely client-driven.

### Pricing & Security
- Server always recalculates totals from DB prices — never trust client amounts
- COD order creation, Razorpay order creation, and Razorpay verification must all use `createCheckoutQuote()` from `lib/checkout/quote.ts`
- Pure money math belongs in `lib/checkout/pricing.ts`; route handlers should not duplicate subtotal/shipping/discount logic
- Razorpay signature verification on payment confirmation (`app/api/razorpay/verify/route.ts`)
- Coupon validation runs server-side with all business rules (expiry, limits, min order, user restrictions)
- Shipping: free ≥ ₹999, else ₹99 (see `lib/constants.ts` for source of truth)
- COD: ₹50 fee, cancellable by the owning customer only while `pending` or `confirmed`; cancellation restores stock and rolls back user metrics

### Bargain AI (Discount Negotiation)
- Checkout component (`components/features/checkout-bargain.tsx`) streams AI chat via `/api/bargain`
- Progressive offer and finalization rules live in `lib/bargain/logic.ts` and have unit tests
- DB-backed bargain eligibility context lives in `lib/bargain/context.ts`; the route should not query product/combo caps inline
- Coupon/session DB writes live in `lib/actions/bargain.ts`; `/api/bargain` should orchestrate streaming only
- Generates ephemeral coupons (prefix `BRG-`, 5-min expiry) stored in `coupons` and `bargainSessions` tables only after persistence succeeds
- Product page AI (`components/features/bargain-ai.tsx`) answers questions locally — no API calls

### Payment & Checkout Flow
Two payment paths in `app/checkout/page.tsx`:
- **COD:** POST `/api/orders` → creates order directly with `paymentStatus="pending"`, adds ₹50 COD fee

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fatelessdev/xilar](https://github.com/fatelessdev/xilar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
