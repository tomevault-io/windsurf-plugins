---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Editorial Standards Platform (formerly OpenSciRank) — a platform for technical evaluation and visibility of scientific journals and academic books. Journals are evaluated against weighted criteria, scored, and may earn an "Editorial Standards Seal" (1-year validity). Books are indexed for a fee.

The master business logic document is `business-logic.md` at the project root.

## Development Environment

This project uses **Laravel Sail** (Docker). All commands must run through Sail:

```bash
./vendor/bin/sail up -d                    # Start containers (app on port 5000)
./vendor/bin/sail artisan migrate          # Run migrations
./vendor/bin/sail artisan db:seed          # Seed all (Admin, Categories, Criteria, Products)
./vendor/bin/sail npm run dev              # Vite dev server with HMR
./vendor/bin/sail artisan test             # Run PHPUnit tests
./vendor/bin/sail artisan test --filter=AuthenticationTest  # Single test
./vendor/bin/sail composer test:lint       # Pint linter
```

**Key URLs in development:**
- App: `http://localhost:5000`
- Filament Admin: `http://localhost:5000/admin`
- Mailpit (email testing): `http://localhost:8025`

**Admin credentials:** `admin@editorialstandards.com` / `password` (seeded by `AdminUserSeeder`, role `super_admin`). No hay usuario "editor" de prueba seedeado — registrar uno vía `/register` para probar flujos de editor.

## Architecture

**Stack:** Laravel 12, Livewire 4, Filament v5, Tailwind CSS 4, Stripe, MySQL, Sail

### Status Machine (Journals)

```
draft → [pay $99] → submitted → [admin evaluates] → evaluated / certified / rejected
                                                    ↘ requires_changes_evaluation → [user corrects, FREE] → submitted
       → [free]   → pending_listing → [admin reviews] → listed / rejected
                                                       ↘ requires_changes_listing → [user corrects, FREE] → pending_listing
listed → [pay $99] → submitted (evaluation flow)
evaluated / certified / rejected → [pay $99 reevaluation] → submitted
certified → seal_status: active → expiring_soon (30d) → expired → status reverts to evaluated
```

Resubmisión tras `requires_changes_*` es **gratuita** (business-logic 16.5). Cada nueva evaluación o re-evaluación sí tiene costo asociado.

Books: `draft → [pay $49] → submitted → pending_listing → listed / rejected / requires_changes_listing → [free resubmit] → pending_listing`

### Scoring Algorithm

`Journal::calculateScore()` — weighted sum of CriteriaItems. If ANY core indicator (`is_core=true`) fails, score is capped at 49%. Seal requires ≥75% AND all 5 critical indicators met. Score shown as percentage (0-100%), no letter levels.

### Payment Flow (Stripe)

1. User selects plan + addons in `PaymentCheckout` Livewire component
2. `StripePaymentService::createCheckoutSession()` creates Stripe session
3. Stripe redirects to `CheckoutSuccessController` (sync verification)
4. `StripeWebhookController` handles `checkout.session.completed` (async)
5. Payment record created, journal/book status updated to `submitted`
6. `PaymentConfirmed` notification sent

Products identified by `slug`: `journal-evaluation`, `journal-reevaluation`, `seal-renewal-1y`, `seal-renewal-2y`, `seal-renewal-3y`, `book-listing`, `book-listing-featured-1y`, `action-plan-consulting`, `new-journal-consulting`. Express service is no longer a public SKU — it is a +$50 uplift toggled at checkout for evaluation/re-evaluation flows. Legacy slugs kept inactive for FK integrity: `express-evaluation`, `institutional-pack`. The old `premium-report` slug was renamed to `action-plan-consulting` (roadmap #17, 2026-05-13).

**Consulting products:** Two SKUs, both create `AdminTask` of type `consulting`.
- `action-plan-consulting` (USD 215): add-on to a journal evaluation. `Payment.payable_type=Journal`. 1 session.
- `new-journal-consulting` (USD 1,500): standalone product for editors creating a new journal. `Payment.payable_type=User` (no journal exists yet). 3 sessions package + domain + OJS hosting for 12 months. "Pack Lanzamiento Editorial".

**Consulting scheduling flow (Sprint 3.6 #39, 2026-05-14):**
1. Payment confirmed → `AdminTask` created (status `pending`) + `ConsultingPaymentConfirmed` email to editor.
2. Evaluator action "Proponer fechas" → 1-3 candidate slots → status `proposal_sent` → email to editor with signed accept-URLs.
3. Editor in `/app/consulting` accepts one slot → status `scheduled` + `.ics` attachment + both parties emailed.
4. Cron `consulting:send-reminders` 24h before sends `ConsultingReminder`.
5. Cron `consulting:expire-proposals` daily expires unanswered proposals after 5 business days.
6. Cancellation policy: >48h reagenda libre, 24-48h una sola vez, <24h pierde la sesión. Override por super_admin con motivo. Tracked en `admin_tasks.reschedule_count` (cap 3 rondas).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aldorodrigo/OpenSciRank](https://github.com/aldorodrigo/OpenSciRank) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
