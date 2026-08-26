---
trigger: always_on
description: This is a modern full-stack monorepo built with:
---

# Cursor AI Rules for your-saas-starterkit

## Project Overview
This is a modern full-stack monorepo built with:
- **Landing**: Astro static site (root domain)
- **Dashboard**: React 19 + Vite + TanStack Router + Tailwind CSS v4 (app subdomain)
- **Backend**: Elysia + Bun + PostgreSQL + Drizzle ORM (app subdomain, `/api`)
- **Monorepo**: Turborepo with Bun workspaces
- **Auth**: Better Auth with email verification & OAuth
- **UI**: shadcn/ui components + Radix UI primitives (a small subset also lives as plain `.astro` components in `apps/landing`)
- **Linting**: Biome only — no ESLint anywhere in this repo

## Code Style & Conventions

### TypeScript
- Always use TypeScript with strict mode
- Avoid `any` types - use `unknown` or proper types
- Prefer type inference over explicit types when obvious
- Use interfaces for object shapes, types for unions
- Export types and interfaces for reusability

### React
- Use function components with hooks exclusively
- Prefer arrow functions for components
- Use TypeScript for component props
- Keep components small and focused (< 200 lines)
- Extract reusable logic into custom hooks
- Use proper TypeScript types for refs and events

### Naming Conventions
- **Components**: PascalCase (e.g., `UserProfile.tsx`)
- **Files**: kebab-case (e.g., `user-profile.tsx`)
- **Functions**: camelCase (e.g., `getUserData`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `API_BASE_URL`)
- **Types/Interfaces**: PascalCase (e.g., `UserProfile`)
- **Hooks**: camelCase with 'use' prefix (e.g., `useUserData`)

### File Organization
```
apps/landing/src/
├── components/     # .astro sections + Analytics.astro (pixels)
├── content/        # landing-content.ts (copy)
├── layouts/        # BaseLayout.astro
└── pages/          # File-based routes

apps/web/src/
├── components/     # Reusable UI components (dashboard only — no marketing sections)
├── routes/         # TanStack Router route components (auth + app screens only)
├── hooks/          # Custom React hooks
├── lib/            # Utilities, auth-client, Eden API client
└── assets/         # Static assets

apps/api/
├── routes/         # Elysia route plugins
├── lib/            # auth.ts (Better Auth config), auth-plugin.ts (Elysia mount), email
├── app.ts          # Elysia app + route composition, exports `App` type
└── index.ts        # Server entry — Host-header static dispatch in production

packages/
├── database/       # Drizzle schemas
├── shared/         # Shared types & schemas
└── config/         # Environment config
```

### Import Order
1. External packages (React, etc.)
2. Internal packages (@your-saas-starterkit/*)
3. Relative imports (./components, etc.)
4. Types (import type {})
5. Assets and styles

Example:
```typescript
import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';
import { Button } from '@/components/ui/button';
import { getUserData } from './api';
import type { User } from '@your-saas-starterkit/shared';
```

## Frontend Specific

### TanStack Router
- Define routes in `src/routes/` directory
- Use file-based routing conventions
- Leverage type-safe navigation
- Use route loaders for data fetching
- Handle loading and error states

### TanStack Query
- Create query functions in separate files
- Use proper query keys (arrays)
- Implement optimistic updates where appropriate
- Handle loading, error, and success states
- Use mutations for write operations

### Styling
- Use Tailwind CSS utility classes exclusively
- Follow shadcn/ui patterns for components
- Use the `cn()` utility for conditional classes
- Avoid inline styles and custom CSS
- Use design tokens from Tailwind config

### Forms
- Use React Hook Form for form management
- Validate with Zod schemas from @your-saas-starterkit/shared
- Show user-friendly error messages
- Implement proper loading states
- Handle form submission errors gracefully

## Backend Specific

### Elysia API
- Define each route group as its own Elysia plugin (`new Elysia().get/post(...)`), then `.use()` it into `apps/api/app.ts` under a `.group('/api', ...)` sub-path
- Return proper HTTP status codes
- Use Elysia's native `t.Object(...)` schema validation, not a separate validator middleware
- Better Auth is mounted as its own Elysia plugin (`lib/auth-plugin.ts`) via `.mount(auth.handler)` — this is the documented framework-integration pattern, not an official `better-auth/elysia` package
- The API is a pure JSON API — it does not serve static files or an SPA fallback; that's handled by Host-header dispatch in `apps/api/index.ts` at the Bun.serve level, outside the Elysia app itself

### Database
- Define all schemas in `packages/database`
- Use Drizzle ORM for type-safe queries
- Prefer transactions for multi-step operations
- Create indexes for frequently queried fields
- Use migrations for schema changes

### Validation
- Define Zod schemas in `packages/shared`
- Validate all API inputs
- Reuse schemas between frontend and backend
- Provide clear validation error messages

### Authentication
- Use Better Auth for user management
- Implement proper session handling
- Secure sensitive routes with middleware
- Handle auth errors gracefully

## Best Practices

### Error Handling
- Always handle errors explicitly
- Provide user-friendly error messages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JuanPabloGilA/your-saas-starterkit](https://github.com/JuanPabloGilA/your-saas-starterkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
