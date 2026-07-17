---
trigger: always_on
description: > **This is the #1 project rule. Violating TDD is NEVER acceptable. No exceptions. No shortcuts.**
---

# Project Guidelines

## Testing Methodology — MANDATORY TDD (Red/Green)

> [!CAUTION]
> **This is the #1 project rule. Violating TDD is NEVER acceptable. No exceptions. No shortcuts.**

### Hard Gates — Enforced at Every Stage

1. **PLANNING gate**: Every implementation plan MUST include a **"Tests (RED)"** section before any **"Implementation (GREEN)"** section. If the plan does not contain test code/descriptions, the plan is incomplete and MUST NOT be approved.

2. **EXECUTION gate**: During execution, test files MUST be created and run (failing) BEFORE writing any implementation code. The sequence is always:
   - Write the test → Run it → Confirm it FAILS (RED)
   - Write the minimal implementation → Run it → Confirm it PASSES (GREEN)
   - Never reverse this order. Never skip the RED step.

3. **VERIFICATION gate**: `npm run test:run` must be run at the end. New test count must be > 0 for any new feature or bug fix.

### Pre-Flight Checklist — STOP Before Every Code Change

> [!CAUTION]
> **Before writing or editing ANY implementation file, ask yourself these questions. If any answer is YES, you MUST write tests FIRST.**

1. Am I adding a new component, function, or helper? → **YES = tests first**
2. Am I changing what gets rendered or displayed? → **YES = tests first**
3. Am I adding conditional logic (if/else, filtering, mapping)? → **YES = tests first**
4. Am I fixing a bug? → **YES = write a test that reproduces the bug first**
5. Does this change affect data flow or user-visible behavior? → **YES = tests first**

**There is NO size exemption.** A "small" change, a "simple" UI tweak, or a "quick" helper function still requires TDD. The size of the change does not excuse skipping the process.

### What Requires Tests

- Every new server action
- Every new API route
- Every new utility/helper function
- Every new UI component (even small ones like pills, badges, indicators)
- Every new rendering helper or sub-component added to an existing component
- Every bug fix (write a test that reproduces the bug first)
- UI behavior changes that affect data flow (e.g., checkbox toggles that call server actions)
- Conditional rendering logic (e.g., showing/hiding elements based on props or state)

## Tech Stack

- **Framework**: Next.js 16 (App Router, Turbopack)
- **Language**: TypeScript, React 19.2.0
- **Styling**: Tailwind CSS v4 + shadcn/ui
- **Database**: Supabase (PostgreSQL)
- **Authentication**: Supabase Auth
- **Payments**: Stripe
- **Validation**: Zod

## Context7 Library IDs

Use these pre-resolved Library IDs when querying Context7 to ensure version compatibility. Run `resolve-library-id` if these are missing or outdated.

| Library          | Version         | Context7 ID                   |
| ---------------- | --------------- | ----------------------------- |
| **Next.js**      | 16 (App Router) | `/vercel/next.js`             |
| **React**        | 19              | `/facebook/react`             |
| **Tailwind CSS** | v4              | `/websites/tailwindcss`       |
| **Supabase**     | Latest          | `/supabase/supabase-js`       |
| **Stripe**       | Latest          | `/stripe/stripe-node`         |
| **Zod**          | v4              | `/colinhacks/zod`             |
| **Shadcn/UI**    | Latest          | `/shadcn-ui/ui`               |
| **Resend**       | Latest          | `/resend/resend-node`         |
| **Zoho Books**   | v3 API          | `/websites/zoho_books_api_v3` |

## Project Structure

```
src/
├── app/                    # Next.js App Router
│   ├── (auth)/            # Login, register pages
│   ├── (dashboard)/       # Role-based dashboards
│   │   ├── parent/        # Parent portal
│   │   ├── teacher/       # Teacher portal
│   │   ├── student/       # Student portal
│   │   ├── admin/         # Admin portal
│   │   └── class-scheduler/ # Class Scheduler portal
│   └── api/               # API routes (checkout, webhooks)
├── components/
│   ├── ui/                # shadcn/ui components
│   ├── admin/             # Admin action components
│   ├── auth/              # Auth forms
│   ├── classes/           # Class & enrollment components
│   ├── dashboard/         # Shared dashboard layout
│   ├── family/            # Family member components
│   ├── payments/          # Payment components
│   └── class-scheduler/   # Class Scheduler components
├── lib/
│   ├── supabase/          # Supabase client config
│   ├── actions/           # Server actions
│   └── validations.ts     # Zod schemas
├── hooks/                 # Custom React hooks
└── types/                 # TypeScript types
```

## Documentation

- [System Requirements](./docs/REGISTRATION_SYSTEM_DESCRIPTION.md)
- [Architecture Decisions](./docs/architecture_decision_document.md)
- [API Specification](./docs/api_planning_document.md)
- [Zoho Integration Flow](./docs/zoho_integration_flow.md)
- [Deployment Guide](./docs/DEPLOYMENT.md)
- [Testing Guide](./docs/TESTING.md)
- [Manual Testing Guide](./docs/MANUAL_TESTING.md)
- [Design System](./docs/DESIGN_SYSTEM.md)
- [Task List](./docs/TASKS.md)

> [!NOTE]
> Skills are available in the `.agent/skills` directory.

## Development Commands

```bash
npm run dev    # Start dev server (Turbopack)
npm run build  # Production build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JimEastburn/class-registration-system](https://github.com/JimEastburn/class-registration-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
