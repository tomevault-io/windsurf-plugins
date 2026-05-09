---
trigger: always_on
description: This project uses the Better Auth Stripe plugin for payment and subscription functionality. Here are the key patterns and configurations:
---

# Better Auth Stripe Integration Guide

This project uses the Better Auth Stripe plugin for payment and subscription functionality. Here are the key patterns and configurations:

## Project Structure

- **Auth Server**: Look for `auth.ts` in the lib directory for server-side Stripe configuration
- **Auth Client**: [lib/auth-client.ts](mdc:lib/auth-client.ts) - Client-side auth utilities with Stripe client
- **Database Schema**: [lib/db/schema.ts](mdc:lib/db/schema.ts) - Contains subscription tables and user Stripe fields

## Server-Side Configuration

### Basic Stripe Plugin Setup
```typescript
import { betterAuth } from "better-auth"
import { stripe } from "@better-auth/stripe"
import Stripe from "stripe"

const stripeClient = new Stripe(process.env.STRIPE_SECRET_KEY!, {
    apiVersion: "2025-02-24.acacia",
})

export const auth = betterAuth({
    plugins: [
        stripe({
            stripeClient,
            stripeWebhookSecret: process.env.STRIPE_WEBHOOK_SECRET!,
            createCustomerOnSignUp: true,
        })
    ]
})
```

### Subscription Plans Configuration
```typescript
// Static plans
subscription: {
    enabled: true,
    plans: [
        {
            name: "basic",
            priceId: "price_1234567890",
            annualDiscountPriceId: "price_1234567890", // optional
            limits: {
                projects: 5,
                storage: 10
            }
        },
        {
            name: "pro",
            priceId: "price_0987654321",
            limits: {
                projects: 20,
                storage: 50
            },
            freeTrial: {
                days: 14,
            }
        }
    ]
}

// Dynamic plans (from database)
subscription: {
    enabled: true,
    plans: async () => {
        const plans = await db.query("SELECT * FROM plans");
        return plans.map(plan => ({
            name: plan.name,
            priceId: plan.stripe_price_id,
            limits: JSON.parse(plan.limits)
        }));
    }
}
```

## Client-Side Configuration

### Auth Client Setup
```typescript
import { createAuthClient } from "better-auth/client"
import { stripeClient } from "@better-auth/stripe/client"

export const client = createAuthClient({
    plugins: [
        stripeClient({
            subscription: true // enables subscription management
        })
    ]
})
```

## Subscription Management Patterns

### Creating Subscriptions
```typescript
// Basic subscription upgrade
await client.subscription.upgrade({
    plan: "pro",
    successUrl: "/dashboard",
    cancelUrl: "/pricing",
    annual: true, // optional: upgrade to annual plan
    referenceId: "org_123", // optional: defaults to user ID
    seats: 5 // optional: for team plans
});

// With error handling
const { error } = await client.subscription.upgrade({
    plan: "pro",
    successUrl: "/dashboard",
    cancelUrl: "/pricing",
});
if(error) {
    alert(error.message);
}
```

### Listing Active Subscriptions
```typescript
const { data: subscriptions } = await client.subscription.list();

// Get the active subscription
const activeSubscription = subscriptions.find(
    sub => sub.status === "active" || sub.status === "trialing"
);

// Check subscription limits
const projectLimit = activeSubscription?.limits?.projects || 0;
```

### Canceling Subscriptions
```typescript
const { data } = await client.subscription.cancel({
    returnUrl: "/account",
    referenceId: "org_123" // optional, defaults to userId
});
```

### Restoring Canceled Subscriptions
```typescript
const { data } = await client.subscription.restore({
    referenceId: "org_123" // optional, defaults to userId
});
```

## Reference System for Organizations

### Team/Organization Subscriptions
```typescript
// Create subscription for organization
await client.subscription.upgrade({
    plan: "team",
    referenceId: "org_123456",
    seats: 10, // team members
    successUrl: "/org/billing/success",
    cancelUrl: "/org/billing"
});

// List organization subscriptions
const { data: subscriptions } = await client.subscription.list({
    query: {
        referenceId: "org_123456"
    }
});
```

### Authorization for Reference IDs
```typescript
subscription: {
    authorizeReference: async ({ user, session, referenceId, action }) => {
        if (action === "upgrade-subscription" || action === "cancel-subscription" || action === "restore-subscription") {
            const org = await db.member.findFirst({
                where: {
                    organizationId: referenceId,
                    userId: user.id
                }   
            });
            return org?.role === "owner"
        }
        return true;
    }
}
```

## Webhook Configuration

### Required Webhook Events
Set up webhooks in Stripe dashboard for:
- `checkout.session.completed`
- `customer.subscription.updated` 
- `customer.subscription.deleted`

Webhook URL: `https://your-domain.com/api/auth/stripe/webhook`

### Custom Event Handling
```typescript
stripe({
    onEvent: async (event) => {
        switch (event.type) {
            case "invoice.paid":
                // Handle paid invoice
                break;
            case "payment_intent.succeeded":

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inboundemail/inbound](https://github.com/inboundemail/inbound) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
