---
trigger: always_on
description: Rules for implementing desktop browser payment flow (Stripe Checkout), transaction persistence, and success page handling separate from mobile workflow. Includes membership subscription flow with Stripe Subscription ID persistence.
---


- **Main Points in Bold**
  - Desktop flow uses Stripe Checkout Sessions (`cs_`) OR Stripe Elements inline (ExpressCheckoutElement + PaymentElement) for payment processing
  - **Stripe Elements Inline Option**: Desktop can use Stripe Elements (`StripeDesktopCheckout` component) to show Apple Pay, Google Pay, Link, and card form directly on the page (no redirect)
  - **Stripe Checkout Session Option**: Desktop can also use Stripe Checkout Sessions (hosted payment page) via redirect
  - Desktop flow persists transactions immediately when payment succeeds via GET endpoint fallback
  - Desktop flow is completely separate from mobile workflow - uses different entry points and detection methods
  - Desktop flow must include both `tenantId` and `paymentMethodDomainId` in all backend requests
  - Desktop flow creates both transaction and transaction items via `createTransactionFromPaymentIntent`
  - **Membership Subscription Flow**: Desktop membership flow creates Stripe Subscription for recurring billing and stores `stripeSubscriptionId`, `stripeCustomerId`, and `stripePaymentIntentId` in database
  - **CRITICAL**: Membership subscriptions must filter out CANCELLED/EXPIRED subscriptions when looking up existing subscriptions
  - Mobile workflow (`/event/ticket-qr` page) remains completely untouched

# Desktop Browser Payment Flow Architecture

This document outlines the comprehensive desktop browser payment flow architecture for the MCEFEE event management application, detailing how desktop payments differ from mobile payments and how transactions are persisted.

## Overview

The desktop payment flow is fundamentally different from the mobile flow due to:
- Different payment methods (Stripe Checkout vs Payment Request Button)
- Different user experience (hosted payment page vs native wallet)
- Different transaction persistence strategy (immediate creation vs polling with fallback)
- Different success page handling (inline display vs redirect to QR page)

## Desktop vs Mobile Flow Comparison

### Desktop Flow (Two Options)

#### Option 1: Stripe Elements Inline (Preferred for Registered Users)
```
User fills form →
Payment Intent created via /api/stripe/payment-intent (or /api/stripe/membership-payment-intent) →
Stripe Elements rendered inline (ExpressCheckoutElement + PaymentElement) →
Apple Pay/Google Pay/Link/Card form shown on same page →
Payment completion →
Redirect to /event/success?pi=pi_xxx (or /membership/success?pi=pi_xxx) →
SuccessClient detects desktop →
Stays on success page →
GET /api/event/success/process?pi=pi_xxx (or /api/membership/success/process?pi=pi_xxx) →
If transaction/subscription not found (webhook delayed) →
Create transaction/subscription immediately via createTransactionFromPaymentIntent/processMembershipSubscriptionFromPaymentIntent →
For events: Create transaction items via createTransactionItemsBulk →
For memberships: Create Stripe Subscription for recurring billing →
Fetch QR code (events) or display subscription confirmation (memberships) →
Display success page with QR inline (events) or subscription details (memberships)
```

#### Option 2: Stripe Checkout Session (Fallback for Visitors)
```
User fills form →
Stripe Checkout Session created →
Hosted payment page →
Payment completion →
Redirect to /event/success?session_id=cs_xxx or ?pi=pi_xxx →
SuccessClient detects desktop →
Stays on success page →
GET /api/event/success/process?pi=pi_xxx →
If transaction not found (webhook delayed) →
Create transaction immediately via createTransactionFromPaymentIntent →
Create transaction items via createTransactionItemsBulk →
Fetch QR code →
Display success page with QR inline
```

### Mobile Flow (Separate - Unchanged)
```
User taps PRB →
Native wallet sheet opens →
Payment Intent created →
Payment confirmed with PI →
Redirect to /event/success?pi=pi_xxx →
SuccessClient detects mobile →
Shows brief success (2 seconds) →
Redirect to /event/ticket-qr?pi=pi_xxx →
Dedicated QR page with POST fallback →
Transaction created via POST endpoint
```

**CRITICAL**: Desktop and mobile flows are completely separate:
- **Desktop**: Uses GET endpoint with transaction creation fallback
- **Mobile**: Uses POST endpoint (via `/event/ticket-qr` page)
- **Entry Points**: Different API routes and client components
- **Mobile workflow files remain untouched**: `/event/ticket-qr` page and related mobile components are not modified

## Desktop Browser Detection Methods

### Client-Side Detection (`SuccessClient.tsx`)

The application uses multiple methods to reliably detect desktop browsers:

#### 1. User Agent Detection (Primary)
```typescript
const mobileRegexMatch = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini|Mobile|mobile|CriOS|FxiOS|EdgiOS/i.test(userAgent);
const platformMatch = /iPhone|iPad|iPod|Android|BlackBerry|Windows Phone/i.test(platform);
```

#### 2. Screen Width Detection (Secondary)
```typescript
const narrowScreenMatch = window.innerWidth <= 768;
const hasMobileUserAgent = mobileRegexMatch || platformMatch;
```

#### 3. Combined Detection Logic
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/giventadevelop) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
