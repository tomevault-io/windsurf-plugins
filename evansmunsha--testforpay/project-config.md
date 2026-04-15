---
trigger: always_on
description: **TestForPay** is a Next.js full-stack platform connecting app developers with paid Google Play beta testers. Two-sided marketplace with integrated payment processing, fraud detection, and role-based access control.
---

# AI Copilot Instructions for TestForPay

## Project Overview

**TestForPay** is a Next.js full-stack platform connecting app developers with paid Google Play beta testers. Two-sided marketplace with integrated payment processing, fraud detection, and role-based access control.

- **Tech Stack**: Next.js 16, React 19, TypeScript, Prisma ORM, PostgreSQL (Prisma Postgres), Stripe, Tailwind CSS, shadcn/ui
- **Key Domains**: Authentication, Job Management, Applications, Payments, Fraud Detection, PWA Support

## Architecture

### Route Groups & Layout Structure
- `app/(auth)/` — Public auth routes (login, signup, verify-email) with minimal centered layout
- `app/(dashboard)/` — Protected dashboard routes requiring authentication via `useAuth()` hook
- `app/api/` — REST API routes organized by domain (auth, jobs, applications, payments, admin, webhooks)
- No middleware.ts exists; auth protection is client-side via `useAuth()` + redirect in layout

### Data Model (Prisma Schema)
**Key Entities:**
- `User` — Roles (DEVELOPER, TESTER, ADMIN); Stripe integration (customerId/accountId); fraud tracking
- `TestingJob` — Apps needing testers; budget calculated as `paymentPerTester * testersNeeded + platformFee (15%)`
- `Application` — Tester applications to jobs; multi-stage verification (PENDING → APPROVED → VERIFIED → TESTING → COMPLETED)
- `Payment` — Escrow and transfer tracking; integrates with Stripe
- `DeviceInfo` — Tester Android device details (model, version)
- `Notification`, `PushSubscription` — Push notifications & PWA

**Fraud System:**
- `FraudLog` — Tracks suspicious activities
- `User.fraudScore` (0-100), `flagged`, `suspended` with `suspendReason`

### API Pattern (Route Handlers)

All API routes follow this pattern:
```typescript
export async function POST/GET(request: Request) {
  try {
    const currentUser = await getCurrentUser()  // JWT from cookie
    if (!currentUser) return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
    
    // Role check (DEVELOPER/TESTER/ADMIN)
    if (currentUser.role !== 'DEVELOPER') {
      return NextResponse.json({ error: 'Forbidden' }, { status: 403 })
    }
    
    // Parse & validate with Zod schema
    const validated = createJobSchema.parse(body)
    
    // Business logic with Prisma
    // Return NextResponse.json()
  } catch (error) {
    if (error instanceof Error && error.name === 'ZodError') {
      return NextResponse.json({ error: 'Invalid input' }, { status: 400 })
    }
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 })
  }
}
```

**Key Points:**
- Use `getCurrentUser()` from `lib/auth.ts` (reads JWT from httpOnly cookie)
- Always validate input with Zod schemas from `lib/validators.ts`
- Include Prisma `select`/`include` to avoid N+1 queries
- Pass metadata to Stripe for tracking (jobId, developerId, etc.)

### Authentication Flow

**Token Management** (`lib/auth.ts`):
- Generate JWT: 7-day expiry, stores `userId`, `email`, `role`
- Set httpOnly cookie via `setAuthCookie()` (secure in production, lax sameSite)
- Get/verify token: `getAuthCookie()` + `verifyToken(token)`
- Current user: `getCurrentUser()` returns `TokenPayload | null`

**Client-Side Auth** (`hooks/use-auth.ts`):
- Fetches `/api/auth/session` on mount (client-side hook marked `'use client'`)
- Returns `user`, `loading`, `isDeveloper`/`isTester`/`isAdmin` computed flags
- Dashboard layout redirects to login if `!user && !loading`

**Note:** No Next.js middleware.ts; protection is layout-level redirect + hook checks.

## Key Patterns & Conventions

### Validation (Zod Schemas)
All input validated in `lib/validators.ts`. Example:
```typescript
export const createJobSchema = z.object({
  appName: z.string().min(3),
  testersNeeded: z.number().min(12).max(500),
  paymentPerTester: z.number().min(5).max(100),
})
export type CreateJobInput = z.infer<typeof createJobSchema>
```
- Use `schema.parse(input)` in API routes (throws on error)
- Reuse types with `z.infer<typeof schema>`

### Stripe Integration (`lib/stripe.ts`)
- **Job Creation Payment**: `createJobPaymentIntent()` creates PaymentIntent; metadata includes jobId, developerId
- **Tester Onboarding**: `createConnectedAccount()` creates Stripe Express account; `createAccountLink()` for onboarding flow
- **Transfers**: `transferToTester()` sends escrowed funds to tester's account
- **Refunds**: `refundPaymentIntent()` issues partial refunds to developer if job is cancelled
- **API Version**: `2025-12-15.clover` (explicitly pinned)

### Job Cancellation with Partial Refunds
Job cancellation (`app/api/jobs/[id]/route.ts`) handles:
1. **Compensation based on progress**:
   - COMPLETED: 100% payment
   - TESTING: 75% payment
   - VERIFIED: 50% payment
   - OPTED_IN: 25% payment
   - APPROVED/PENDING: 0% (no work done)
2. **Payment processing**:
   - Updates Payment records with compensation amounts
   - Sets status to `REFUNDED` for audit trail
   - Calculates platform fee (15%) on actual payouts
3. **Developer refund**:
   - Calls `refundPaymentIntent()` for unused funds

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/evansmunsha) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
