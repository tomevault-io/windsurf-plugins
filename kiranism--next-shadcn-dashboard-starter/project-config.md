---
trigger: always_on
description: This file provides essential information for AI coding agents working on this project. It contains project-specific details, conventions, and guidelines that complement the README.
---

# AGENTS.md - AI Coding Agent Reference

This file provides essential information for AI coding agents working on this project. It contains project-specific details, conventions, and guidelines that complement the README.

---

## Project Overview

**Next.js Admin Dashboard Starter** is a production-ready admin dashboard template built with:

- **Framework**: Next.js 16 (App Router)
- **Language**: TypeScript 5.7
- **Styling**: Tailwind CSS v4
- **UI Components**: shadcn/ui (New York style)
- **Authentication**: Clerk (with Organizations/Billing support)
- **Error Tracking**: Sentry
- **Charts**: Recharts
- **Containerization**: Docker (Node.js & Bun Dockerfiles)
- **Package Manager**: Bun (preferred) or npm

The project follows a feature-based folder structure designed for scalability in SaaS applications, internal tools, and admin panels.

---

## Technology Stack Details

### Core Framework & Runtime

- Next.js 16.0.10 with App Router
- React 19.2.0
- TypeScript 5.7.2 with strict mode enabled

### Styling & UI

- Tailwind CSS v4 (using `@import 'tailwindcss'` syntax)
- PostCSS with `@tailwindcss/postcss` plugin
- shadcn/ui component library (Radix UI primitives)
- CSS custom properties for theming (OKLCH color format)

### State Management

- Zustand 5.x for local UI state in the stateful demo features
- Nuqs for URL search params state management
- TanStack Form + Zod for form handling (via `useAppForm` hook)

### Data Fetching & Caching

- TanStack React Query for data fetching, caching, and mutations
- Server-side prefetching with `HydrationBoundary` + `dehydrate`
- Client-side `useQuery` + nuqs `shallow: true` for tables (no RSC round-trips on pagination/filter)
- `useMutation` + `invalidateQueries` for form submissions
- Query client singleton in `src/lib/query-client.ts`

### Authentication & Authorization

- Clerk for authentication and user management
- Clerk Organizations for multi-tenant workspaces
- Clerk Billing for subscription management (B2B)
- Client-side RBAC for navigation visibility

### Data & APIs

- TanStack Table for data tables
- TanStack React Query for data fetching and mutations
- Recharts for analytics/charts
- Service layer per feature (`api/types.ts` → `api/service.ts` → `api/queries.ts`)
- Route handlers at `src/app/api/` (for Route Handler or BFF patterns)
- Mock data in `src/constants/mock-api*.ts` (default, swap via service layer)
- API client utility in `src/lib/api-client.ts` (for fetch-based patterns)

### Development Tools

- ESLint 8.x with Next.js core-web-vitals config
- Prettier 3.x with prettier-plugin-tailwindcss
- Husky for git hooks
- lint-staged for pre-commit formatting

---

## Project Structure

```
/src
├── app/                    # Next.js App Router
│   ├── auth/              # Authentication routes (sign-in, sign-up)
│   ├── dashboard/         # Dashboard routes
│   │   ├── overview/      # Parallel routes (@area_stats, @bar_stats, etc.)
│   │   ├── product/       # Product management pages
│   │   ├── kanban/        # Kanban board page
│   │   ├── chat/          # Messaging page
│   │   ├── notifications/ # Notifications page
│   │   ├── workspaces/    # Organization management
│   │   ├── billing/       # Subscription billing
│   │   ├── exclusive/     # Pro plan feature example
│   │   └── profile/       # User profile
│   ├── api/               # API routes (if any)
│   ├── layout.tsx         # Root layout with providers
│   ├── page.tsx           # Landing page
│   ├── global-error.tsx   # Global error boundary
│   └── not-found.tsx      # 404 page
│
├── components/
│   ├── ui/                # shadcn/ui components (50+ components)
│   ├── layout/            # Layout components (sidebar, header, etc.)
│   ├── forms/             # Form field wrappers
│   ├── themes/            # Theme system components
│   ├── kbar/              # Command+K search bar
│   ├── icons.tsx          # Icon registry
│   └── ...
│
├── features/              # Feature-based modules
│   ├── auth/              # Authentication components
│   ├── overview/          # Dashboard analytics
│   ├── products/          # Product management (React Query + nuqs)
│   │   ├── api/
│   │   │   ├── types.ts   # Type contract (response shapes, filters, payloads)
│   │   │   ├── service.ts # Data access layer (swap for your backend)
│   │   │   └── queries.ts # React Query options + key factories
│   │   ├── components/    # Listing, form, table components
│   │   ├── schemas/       # Zod schemas
│   │   └── constants/     # Filter options
│   ├── users/             # User management (React Query + nuqs)
│   │   ├── api/           # Same pattern: types.ts → service.ts → queries.ts
│   │   └── components/    # Listing, table components
│   ├── react-query-demo/  # React Query showcase (Pokemon API)
│   ├── kanban/            # Kanban board with dnd-kit
│   ├── chat/              # Messaging UI (conversations, bubbles, composer)
│   ├── notifications/     # Notification center & store
│   └── profile/           # Profile management
│
├── config/                # Configuration files
│   ├── nav-config.ts      # Navigation with RBAC
│   └── ...
│

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kiranism/next-shadcn-dashboard-starter](https://github.com/Kiranism/next-shadcn-dashboard-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
