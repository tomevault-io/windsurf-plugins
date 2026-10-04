---
trigger: always_on
description: This file provides essential information for AI coding agents working on this project. It contains project-specific details, conventions, and guidelines that complement the README.
---

# AGENTS.md - AI Coding Agent Reference

This file provides essential information for AI coding agents working on this project. It contains project-specific details, conventions, and guidelines that complement the README.

---

## Project Overview

**DevOne** is a self-hosted developer workspace built with:

- **Framework**: Next.js 16 (App Router)
- **Language**: TypeScript 5.7
- **Styling**: Tailwind CSS v4
- **UI Components**: shadcn/ui (New York style)
- **Charts**: Recharts
- **Containerization**: Docker (Node.js & Bun Dockerfiles)
- **Package Manager**: Bun (preferred) or npm

The project follows a feature-based folder structure. The current foundation provides a dashboard shell; product integrations are added as independently testable slices.

---

## Technology Stack Details

### Core Framework & Runtime

- Next.js 16.2.12 with App Router
- React 19.2.4
- TypeScript 5.7.2 with strict mode enabled

### Styling & UI

- Tailwind CSS v4 (using `@import 'tailwindcss'` syntax)
- PostCSS with `@tailwindcss/postcss` plugin
- shadcn/ui component library (Radix UI primitives)
- CSS custom properties for theming (OKLCH color format)

### State Management

- Nuqs for URL search params state management
- TanStack Form + Zod for form handling (`createFormHook` + shadcn `Field`-anatomy components)

### Data Fetching & Caching

- TanStack React Query for data fetching, caching, and mutations
- Server-side prefetching with `HydrationBoundary` + `dehydrate`
- Client-side `useQuery` + nuqs `shallow: true` for tables (no RSC round-trips on pagination/filter)
- `useMutation` + `invalidateQueries` for form submissions
- Query client singleton in `src/lib/query-client.ts`

### Data & APIs

- TanStack Table for data tables
- TanStack React Query for data fetching and mutations
- Recharts for analytics/charts
- Service layer per feature (`api/types.ts` → `api/service.ts` → `api/queries.ts`)
- Route handlers at `src/app/api/` (for Route Handler or BFF patterns)
- Mock data in `src/constants/mock-api*.ts` (default, swap via service layer)
- API client utility in `src/lib/api-client.ts` (for fetch-based patterns)

### Development Tools

- OxLint for linting
- Oxfmt for formatting
- Husky for git hooks
- lint-staged for pre-commit formatting

---

## Project Structure

```
/src
├── app/                    # Next.js App Router
│   ├── dashboard/         # Dashboard routes
│   │   ├── overview/      # DevOne dashboard
│   │   ├── [section]/     # Placeholder product-area routes
│   ├── api/               # API routes (if any)
│   ├── layout.tsx         # Root layout with providers
│   ├── page.tsx           # Landing page
│   ├── global-error.tsx   # Global error boundary
│   └── not-found.tsx      # 404 page
│
├── components/
│   ├── ui/                # shadcn/ui components (50+ components)
│   ├── layout/            # Layout components (sidebar, header, etc.)
│   ├── forms/             # Field components (shadcn TanStack Form anatomy) + demos
│   ├── themes/            # Theme system components
│   ├── kbar/              # Command+K search bar
│   ├── icons.tsx          # Icon registry
│   └── ...
│
├── features/              # Feature modules added as product slices
│
├── config/                # Configuration files
│   ├── nav-config.ts      # Navigation with RBAC
│   └── ...
│
├── hooks/                 # Custom React hooks
│   ├── use-nav.ts         # RBAC navigation filtering
│   ├── use-data-table.ts  # Data table state
│   └── ...
│
├── lib/                   # Utility functions
│   ├── utils.ts           # cn() and formatters
│   ├── searchparams.ts    # Search param utilities
│   └── ...
│
├── types/                 # TypeScript type definitions
│   └── index.ts           # Core types (NavItem, etc.)
│
└── styles/                # Global styles
    ├── globals.css        # Tailwind imports + view transitions
    ├── theme.css          # Theme imports
    └── themes/            # Individual theme files

/docs                      # Documentation
│   └── themes.md          # Theme customization guide

/scripts                   # Runnable project checks

Dockerfile                 # Node.js production Dockerfile
Dockerfile.bun             # Bun production Dockerfile
.dockerignore              # Docker build exclusions
```

---

## Build & Development Commands

```bash
# Install dependencies
bun install

# Development server
bun run dev          # Starts at http://localhost:3000

# Build for production
bun run build

# Start production server
bun run start

# Linting
bun run lint         # Run ESLint
bun run lint:fix     # Fix ESLint issues and format
bun run lint:strict  # Zero warnings tolerance

bun run typecheck    # tsc --noEmit

# Formatting
bun run format       # Format with Prettier
bun run format:check # Check formatting

# Git hooks
bun run prepare      # Install Husky hooks
```

---

## Environment Configuration

Copy `env.example.txt` to `.env.local` and configure:

## Code Style Guidelines

### TypeScript

- Strict mode enabled
- Use explicit return types for public functions
- Prefer interface over type for object definitions
- Use `@/*` alias for imports from src


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imnotseanwtf/DevOne](https://github.com/imnotseanwtf/DevOne) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
