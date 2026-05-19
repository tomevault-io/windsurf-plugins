---
trigger: always_on
description: Airweave supports two ways to pay for subscription plans while always metering usage monthly:
---

## Airweave Stripe Billing Rules

### Overview
Airweave supports two ways to pay for subscription plans while always metering usage monthly:
- Monthly subscription (normal Stripe subscription, monthly billing)
- Yearly prepay with monthly usage (customer prepays a discounted annual amount, we apply a 20% coupon for 12 months, draw down from Stripe customer balance monthly; after 12 months, coupon expires and subscription defaults to standard monthly pricing of the same plan)

Plans: `developer` (free, $0 subscription), `pro`, `team`, `enterprise` (handled outside Stripe). Only `pro` and `team` support yearly prepay.

Key principles:
- Upgrades apply immediately with proration (never wait), so customers can use higher limits right away
- Downgrades are scheduled at the end of the billing period (monthly or yearly if on yearly prepay)
- Yearly prepay is implemented via one-time payment → credit balance + 20% coupon for 12 months → normal monthly after expiry


### Architecture
- API endpoints: `airweave/api/v1/endpoints/billing.py`
- Service orchestrator: `airweave/billing/service.py`
- Business rules: `airweave/billing/plan_logic.py`
- DB transactions (periods/usage/records): `airweave/billing/transactions.py`
- Stripe client (SDK wrapper): `airweave/integrations/stripe_client.py`
- Webhook processor: `airweave/billing/webhook_handler.py`
- Frontend flows: `frontend/src/pages/Onboarding.tsx`, `frontend/src/components/settings/BillingSettings.tsx`

Data model highlights:
- Organization billing state stored in `OrganizationBilling` with fields like `billing_plan`, `stripe_customer_id`, `stripe_subscription_id`, `has_yearly_prepay`, `pending_plan_change`, `current_period_start/end`, etc.
- Billing periods tracked in `BillingPeriod` with status transitions (ACTIVE → COMPLETED, GRACE, ENDED_UNPAID) and associated `Usage` rows.


### Endpoints
- POST `/billing/checkout-session` → Monthly subscription checkout (Stripe Subscription mode). Requires plan and success/cancel URLs.
- POST `/billing/yearly/checkout-session` → Yearly prepay checkout (Stripe Payment mode). Creates a one-time checkout for the full-year prepaid amount, records intent, and a 12-month 20% coupon to be applied post-payment.
- POST `/billing/update-plan` → Update plan and optionally the period (`monthly`|`yearly`). Business rules ensure upgrades are immediate and downgrades are scheduled.
- GET `/billing/subscription` → Returns `SubscriptionInfo`: plan, status, period boundaries, limits, yearly flags, pending plan changes, etc.
- POST `/billing/cancel` → Cancel at period end.
- POST `/billing/reactivate` → Clear cancellation.
- POST `/billing/cancel-plan-change` → Clear a scheduled plan change.
- POST `/billing/portal-session` → Stripe Customer Portal session.
- POST `/billing/webhook` → Stripe webhook receiver (signature-verified).

Auth/Context: Read endpoints (e.g. `GET /billing/subscription`) use `Depends(deps.get_context)`. Write endpoints use `deps.require_org_role(logic.can_manage_billing)` to enforce admin/owner role checks — except `POST /billing/webhook`, which is unauthenticated and instead verified via `stripe_client.verify_webhook_signature` using `STRIPE_WEBHOOK_SECRET`.


### Monthly vs Yearly Prepay
Monthly:
- Standard Stripe subscription with the plan’s monthly price
- On upgrades: switch price immediately with proration; on downgrades: schedule price change for period end

Yearly Prepay:
- One-time payment for 12 months at 20% discount
- We create/get a 20% coupon (12 months, repeating) and apply it to the subscription
- We credit the customer balance by the prepaid amount; monthly invoices draw from this credit
- After 12 months: coupon expires and plan continues as standard monthly at regular price (no coupon). DB yearly flags are cleared on/after renewal following expiry

Amounts (cents):
- PRO yearly prepay: `12 * $20.00 * 0.8 = 19200`
- TEAM yearly prepay: `12 * $299.00 * 0.8 = 287040`


### Core Business Rules
Defined in `plan_logic.py` and enforced by `service.py`:

- Change type: `UPGRADE` (immediate), `DOWNGRADE` (scheduled), `SAME`, `REACTIVATION`
- Payment method required to move into paid plans; otherwise return a message instructing to use checkout
- On monthly:
  - Upgrade: immediate subscription update (proration on Stripe)
  - Downgrade: schedule by setting `pending_plan_change` and effective date at current period end
- On yearly prepay:
  - Upgrades within yearly prepay: immediate price update; coupon retained or updated as needed, and additional credit may be added for plan upgrades to next yearly tier
  - Downgrades while yearly prepay is active: schedule for yearly expiry (`pending_plan_change` at `yearly_prepay_expires_at`)
  - Switching from yearly → monthly of same plan: automatic at yearly expiry (we surface as pending change for UI)
  - Switching from yearly → higher monthly (e.g., pro yearly → team monthly): remove discount and update price immediately
  - Disallowed: direct cross-year downgrades (e.g., team yearly → pro yearly). UX hints recommend first landing on monthly at expiry, then switching to yearly


### Orchestration Flows

Organization creation:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airweave-ai/airweave](https://github.com/airweave-ai/airweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
