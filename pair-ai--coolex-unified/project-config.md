---
trigger: always_on
description: HVAC equipment selection platform for COOLEX (by RIC). Engineers and dealers use it to:
---

# COOLEX Unified Selector

## Project Overview

HVAC equipment selection platform for COOLEX (by RIC). Engineers and dealers use it to:
1. Select HVAC equipment through a 7-step guided wizard
2. Manage projects with units, revisions, and submittals
3. Generate PDF submittal documents
4. Admin: manage users and pricing

**Current state**: Phase 1 complete — fully functional frontend with mock data. No real backend, database, or authentication yet.

**Demo credentials** (any password works):
- `admin@coolex.com` — admin role (sees /admin routes, pricing in options)
- `engineer@coolex.com` — engineer role
- `dealer@coolex.com` — dealer role (pricing hidden)

## Tech Stack

- **Next.js 16.1.6** (App Router, Turbopack) with **React 19.2.3**
- **TypeScript 5** (strict mode)
- **Tailwind CSS v4** — CSS-first config via `@tailwindcss/postcss` plugin, NOT v3
- **shadcn/ui** — Radix primitives in `components/ui/`, configured via `components.json`
- **Zustand 5** — global state (5 stores, persisted to localStorage)
- **TanStack Query v5** — data fetching/caching layer
- **React Hook Form 7 + Zod 4** — form state and validation
- **Framer Motion 12** — animations
- **@react-pdf/renderer 4** — client-side PDF generation
- **Lucide React** — icons

## Commands

```bash
npm run dev      # Start dev server (Turbopack) → http://localhost:3000
npm run build    # Production build
npm run start    # Serve production build
npm run lint     # ESLint (Next.js core-web-vitals + TypeScript)
```

No test framework is set up yet. No `.env` files needed — all data is mocked.

## Architecture

```
coolex-unified/
├── app/                          # Next.js App Router
│   ├── layout.tsx                # Root layout (fonts, metadata, Providers)
│   ├── page.tsx                  # Root redirect
│   ├── providers.tsx             # QueryClient + TooltipProvider (client)
│   ├── globals.css               # Tailwind v4 theme, CSS vars, brand colors
│   ├── (auth)/                   # Unauthenticated routes
│   │   ├── login/page.tsx
│   │   └── register/page.tsx
│   ├── (app)/                    # Authenticated app shell (Sidebar + TopBar)
│   │   ├── layout.tsx            # Auth guard, sidebar layout
│   │   ├── dashboard/page.tsx
│   │   ├── projects/page.tsx
│   │   ├── projects/[id]/page.tsx
│   │   └── select/page.tsx       # 7-step selection wizard
│   ├── (admin)/                  # Admin-only (RoleGuard)
│   │   ├── layout.tsx
│   │   └── admin/{users,pricing}/page.tsx
│   └── api/mock/                 # Mock API endpoints (GET only)
│       ├── product-groups/route.ts
│       ├── product-series/route.ts
│       ├── results/route.ts
│       ├── options/route.ts
│       ├── projects/route.ts
│       └── users/route.ts
├── components/
│   ├── ui/                       # shadcn/ui primitives (button, dialog, table, etc.)
│   ├── layout/                   # Sidebar, TopBar, RoleGuard, CommandPalette
│   ├── auth/                     # LoginForm, RegisterForm
│   ├── dashboard/                # StatsCards, RecentProjectsList
│   ├── selection/                # 7-step wizard components
│   │   ├── SelectionStepper.tsx  # Step indicator + navigation
│   │   ├── ProjectInfoForm.tsx   # Step 1
│   │   ├── ProductGroupGrid.tsx  # Step 2
│   │   ├── SeriesGrid.tsx        # Step 3
│   │   ├── DesignConditionsForm.tsx # Step 4
│   │   ├── ResultsTable.tsx      # Step 5
│   │   ├── OptionsConfigurator.tsx # Step 6
│   │   └── SubmittalPreview.tsx  # Step 7
│   ├── projects/                 # ProjectListTable, RevisionHistoryPanel, SelectionSheet*
│   ├── submittal/                # SubmittalPDF, CombinedSubmittalPDF (@react-pdf)
│   └── admin/                    # PriceListManager, UserManagementTable
├── lib/
│   ├── stores/                   # Zustand stores
│   │   ├── auth-store.ts         # Login/logout, user state
│   │   ├── selection-store.ts    # 7-step wizard state machine
│   │   ├── projects-store.ts     # Projects CRUD, units, revisions
│   │   ├── ui-store.ts           # Sidebar state, theme
│   │   └── unit-store.ts         # Imperial/metric toggle
│   ├── mock-data/                # Hardcoded product + user data
│   │   ├── product-groups.ts     # 8 product groups
│   │   ├── product-series.ts     # 16 series
│   │   ├── models.ts             # Router: seriesId → model array
│   │   ├── ngw-models.ts         # NGW fan coil units (10 models)
│   │   ├── acsc-models.ts        # ACSC air-cooled screw chillers (34 models)
│   │   ├── {chcc,ngcc,pngc,pngf}-models.ts  # Other product model files
│   │   ├── options.ts            # Equipment options/accessories
│   │   ├── projects.ts           # Sample projects
│   │   ├── users.ts              # Mock user accounts
│   │   └── countries.ts          # Country list
│   ├── nomenclature.ts           # Model number decoder engine
│   ├── utils.ts                  # cn() — clsx + tailwind-merge
│   └── utils/
│       ├── capacity.ts           # Capacity calculation helpers
│       ├── pdf.ts                # PDF generation utilities
│       └── unit-conversions.ts   # Imperial ↔ metric conversions
├── hooks/                        # React Query hooks
│   ├── useProjects.ts
│   ├── useSelection.ts           # useModels, useOptions, useProductGroups, etc.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pair-AI/coolex-unified](https://github.com/Pair-AI/coolex-unified) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
