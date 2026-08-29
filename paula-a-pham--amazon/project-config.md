---
trigger: always_on
description: Full-stack Amazon clone e-commerce web application with product catalog, cart, checkout, orders, user accounts, reviews/ratings, search, wishlist, seller dashboard, and recommendations.
---

# Amazon Clone — Project Guide

## Project Overview

Full-stack Amazon clone e-commerce web application with product catalog, cart, checkout, orders, user accounts, reviews/ratings, search, wishlist, seller dashboard, and recommendations.

## Tech Stack

| Layer          | Technology                          |
| -------------- | ----------------------------------- |
| Frontend       | React 18+ with TypeScript           |
| Styling        | Tailwind CSS                        |
| State Mgmt     | Zustand (client state) + TanStack Query (server state) |
| Backend        | Node.js with Fastify                |
| Database       | PostgreSQL                          |
| ORM            | Prisma                              |
| Auth           | JWT (access + refresh token rotation) |
| Testing        | Vitest + React Testing Library + Playwright |
| Package Mgr    | pnpm (with workspaces)              |
| Validation     | Zod                                 |
| Build Tool     | Vite (frontend)                     |

## Project Structure (Monorepo)

```
amazon/
├── CLAUDE.md
├── pnpm-workspace.yaml
├── package.json              # Root scripts, shared devDependencies
├── tsconfig.base.json        # Shared TypeScript config
├── .env.example
├── .gitignore
├── docs/
│   ├── api-routes.md         # Full API route reference
│   └── db-schema.md          # Database schema reference
├── packages/
│   └── shared/               # Shared types, constants, validation schemas
│       ├── src/
│       │   ├── types/        # Shared TypeScript interfaces/types
│       │   ├── constants/    # Shared constants (status codes, enums)
│       │   └── validators/   # Zod schemas shared between client & server
│       ├── package.json
│       └── tsconfig.json
├── client/                   # React frontend
│   ├── public/
│   ├── src/
│   │   ├── assets/
│   │   ├── components/       # Reusable UI components
│   │   │   ├── ui/           # Primitives (Button, Input, Modal, etc.)
│   │   │   ├── layout/       # Header, Footer, Sidebar, etc.
│   │   │   └── features/     # Feature-specific components
│   │   ├── hooks/            # Custom React hooks
│   │   ├── pages/            # Route-level page components
│   │   ├── stores/           # Zustand stores
│   │   ├── services/         # API client functions (fetch/axios wrappers)
│   │   ├── utils/            # Frontend utility functions
│   │   ├── types/            # Frontend-only types
│   │   ├── App.tsx
│   │   ├── main.tsx
│   │   └── router.tsx        # React Router config
│   ├── e2e/                  # Playwright E2E tests
│   ├── index.html
│   ├── vite.config.ts
│   ├── tailwind.config.ts
│   ├── postcss.config.js
│   ├── tsconfig.json
│   └── package.json
├── server/                   # Fastify backend
│   ├── src/
│   │   ├── plugins/          # Fastify plugins (auth, cors, etc.)
│   │   ├── routes/           # Route handlers grouped by domain
│   │   │   ├── auth/
│   │   │   ├── products/
│   │   │   ├── cart/
│   │   │   ├── orders/
│   │   │   ├── reviews/
│   │   │   ├── users/
│   │   │   ├── wishlist/
│   │   │   ├── sellers/
│   │   │   └── search/
│   │   ├── services/         # Business logic layer
│   │   ├── middlewares/       # Auth guards, validation, rate limiting
│   │   ├── utils/            # Server utility functions
│   │   ├── types/            # Server-only types
│   │   ├── config/           # Environment config, constants
│   │   └── app.ts            # Fastify app setup
│   ├── prisma/
│   │   ├── schema.prisma
│   │   ├── migrations/
│   │   └── seed.ts           # Database seeding
│   ├── tests/                # Vitest unit/integration tests
│   ├── tsconfig.json
│   └── package.json
└── docker-compose.yml        # PostgreSQL + Redis (optional) for local dev
```

## Coding Conventions

### General

- **Language**: TypeScript everywhere — strict mode enabled, no `any` types
- **Formatting**: Use Prettier (default config) + ESLint
- **Imports**: Use path aliases (`@/components`, `@/hooks`, `@/services`, etc.)
- **Naming**:
  - Files/folders: `kebab-case` (e.g., `product-card.tsx`, `auth-service.ts`)
  - Components: `PascalCase` (e.g., `ProductCard`, `CartDrawer`)
  - Functions/variables: `camelCase`
  - Types/Interfaces: `PascalCase`, no `I` prefix (e.g., `User`, not `IUser`)
  - Constants: `UPPER_SNAKE_CASE`
  - Database tables: `snake_case` (Prisma maps to PascalCase models)
- **Exports**: Named exports only — no default exports (except pages)

### Frontend (React)

- Functional components only with arrow functions
- Props type defined inline or with a `Props` suffix (e.g., `ProductCardProps`)
- Use Zustand stores for global client state (cart, UI, auth user)
- Use TanStack Query for all server data fetching and caching
- Tailwind CSS for all styling — no CSS modules or styled-components
- Component structure: UI primitives in `components/ui/`, composed into feature components
- React Router v6+ with lazy loading for route-level code splitting
- **Responsive design**: Mobile-first approach — design for small screens first, scale up with Tailwind breakpoints (`sm:`, `md:`, `lg:`, `xl:`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paula-a-pham/amazon](https://github.com/paula-a-pham/amazon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
