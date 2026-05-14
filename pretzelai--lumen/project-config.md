---
trigger: always_on
description: **What**: A B2B SaaS billing system to support complex pricing models. Startup of 2 people focussed on shipping fast.
---

# Lumen Development Guide

## 1. Project Overview

**What**: A B2B SaaS billing system to support complex pricing models. Startup of 2 people focussed on shipping fast.

**Architecture**: Monorepo (`api`, `ui`, `common`) with Hono/TypeScript API, Next.js UI, Drizzle ORM, PostgreSQL

**Commands**: `bun run dev`, `dev:api`, `dev:ui`, `docker:up`, `docker:down`, `lint`, `format`

## 2. Development Guidelines

### 2.1 Code Style

- TypeScript with strict typing and explicit return types
- camelCase for variables/functions, PascalCase for components
- Double quotes for strings, required semicolons
- Group imports (external, internal, types) with empty line between groups
- Use path aliases: `@/common/...` not `@lumen/common/...`

### 2.2 Security Patterns

- **Critical**: Always use authenticated merchantId from context for access control
- Pattern: `const merchantId = c.get("merchant")?.id; if (!merchantId) return c.json({ error: "Unauthorized" }, 401);`
- Never allow client-provided merchantIds to override auth context
- Merchants must only access their own data; all multi-tenant queries must include merchantId filter

## 3. Core Subsystems

### 3.1 API Security

- Authentication: Session-based (UI), API keys (external), Trigger.dev tokens (jobs)
- API Key Types: Secret (`sk_{env}_{random}`), Publishable (`pk_{env}_{random}`)
- Secret keys are hashed (SHA-256); publishable keys stored as plaintext
- Redis-based rate limiting with sliding window

### 3.2 Plan Versioning

- Append-only: New versions created instead of modifying existing ones
- Existing customers remain on current plan version until explicitly migrated
- Version Intent: Clean audit trail, not multiple active plan variations

### 3.2.1 Plan Creation API

- **Comprehensive Creation**: Plans can be created with metrics, features, and prices in a single transaction
- **New Features**: Can create usage-based features with automatic metric creation and linkage
- **Existing Features**: Can attach existing features to new plans via featureIds
- **Pricing**: Supports monthly and yearly pricing creation during plan creation
- **Transaction Safety**: All operations (metrics, features, plan, prices) happen in a single DB transaction

### 3.3 Feature System

- Feature types: boolean (toggles), numeric (limits), string (settings)
- Many-to-many relationships: features-plans, features-subscriptions
- Denormalized fields for performance (merchantId, customerId, featureSlug)
- Priority: Subscription override → Plan feature → Default value

### 3.4 Subscription System

- Version-based with `stableSubscriptionId` across versions
- Previous versions soft-deleted with `deletedAt` timestamps
- Features inherited from plans with override capability

### 3.5 Billing System

- Price DSL with fixed, per-unit, and usage components
- Recurrence Rules: RFC5545 RRULE format (e.g., `RRULE:FREQ=MONTHLY;BYMONTHDAY=1`)
- Supports fixed-day and anniversary billing with DST awareness
- Smart handling for month-end dates (29-31) and leap years
- PDF generation via React-PDF, stored in S3 with presigned URLs
- Jobs pipeline via Trigger.dev: check billing due → generate invoices → create PDFs
- Multiple Price Components: Support for multiple components in a price with different billing cycles
  - Types: fixed, per_unit, and usage
  - Each component can have its own recurrence rule (monthly, weekly, daily)
  - Components are billed independently based on their own billing cycles
  - Each component generates its own subscription details and invoices

#### 3.5.0 Dunning & Grace Period System

- **Grace Period Management**: Subscriptions enter `past_due` status when payment fails or payment link is created
- **Automatic Flow (Pricing Table)**: Payment failures trigger immediate dunning via webhook → subscription marked `past_due` with grace period
- **Manual Flow (Admin-Created)**: Payment link generation automatically marks subscription `past_due` with grace period
- **Grace Period Timeline**:
  - Day 0: Payment fails or link sent (invoice email already delivered)
  - Day 6 (T-24h): "Grace ending soon" reminder email sent automatically
  - Day 7: Grace expires, configured action enforced (pause/cancel/mark_as_unpaid)
- **Dunning Configuration**: Hierarchical (plan-specific → merchant default → system default)
- **Recovery**: Successful payment automatically resumes subscription and resets billing cycle
- **Jobs Pipeline**:
  - `retryDunningPayments` (daily at 10 AM UTC): Retry failed payments
  - `notifyGraceEndingSoon` (hourly): Send T-24h reminder emails
  - `applyScheduledChanges` (every minute): Enforce expired grace periods

#### 3.5.1 Taxation Scope (B2B vs B2C)

- Current scope: B2B only. We require business identifiers to compute tax (e.g. VAT ID or business name).
- If a customer lacks business identifiers (no `taxId` and no `businessName`), we treat the transaction as B2C and return zero tax.
- Invoices will include a zero-tax line (when applicable) with reason code "B2C Not Supported" for auditability.
- Reason code added: `b2c_tax_not_supported` (code 21) to clearly flag B2C cases.
- First-time and trial-to-paid flows store tax metadata in payment/invoice; renewals re-calculate taxes using the engine.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pretzelai/lumen](https://github.com/pretzelai/lumen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
