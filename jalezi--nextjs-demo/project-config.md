---
trigger: always_on
description: This is a **comprehensive Next.js 16.0.0 App Router demonstration** showcasing different rendering strategies, advanced routing patterns, Server Actions, API Routes with background processing, streaming patterns, error handling, and modern React 19 features. The codebase is educational and demonstrates patterns rather than implementing business logic.
---

# Next.js Demo - AI Coding Agent Instructions

## Project Overview

This is a **comprehensive Next.js 16.0.0 App Router demonstration** showcasing different rendering strategies, advanced routing patterns, Server Actions, API Routes with background processing, streaming patterns, error handling, and modern React 19 features. The codebase is educational and demonstrates patterns rather than implementing business logic.

**Important**: All application code is located in `src/app/` directory following Next.js best practices.

## Key Architecture Patterns

### Rendering Strategy Architecture

The app demonstrates 4 distinct rendering patterns with specific use cases:

- **Static (`/static`)**: Default Next.js caching, pre-rendered at build time
- **Dynamic (`/dynamic`)**: `export const dynamic = "force-dynamic"` + `cache: "no-store"`
- **ISR (`/isr`, `/posts`)**: `export const revalidate = 60` for time-based regeneration  
- **SSG with Params (`/ssg/[id]`)**: `generateStaticParams()` for pre-rendered dynamic routes

### Advanced Routing Patterns

**Parallel Routes (`/parallel/`)**: Layout accepts multiple slot props (`team`, `analytics`) rendered simultaneously using `@team/` and `@analytics/` directories.

**Intercepting Routes (`/photos/`)**: Modal pattern where:
- `@modal/(.)[id]/page.tsx` intercepts navigation to show modal overlay
- `[id]/page.tsx` handles direct URL access/refresh as full page
- Layout renders `{children}` and `{modal}` together

**Server Actions (`/server-actions/`)**: Form submissions and mutations using `"use server"`:
- `actions.ts` exports server functions with `revalidatePath()` for cache invalidation
- Client components use `useFormState` and `useFormStatus` for progressive enhancement
- Demonstrates validation, error handling, and optimistic updates

**Streaming (`/streaming/`)**: Progressive loading with Suspense boundaries:
- Nested `<Suspense>` components with custom `LoadingSkeleton` fallbacks
- `FastComponent` vs `SlowComponent` demonstrate loading prioritization
- Progressive form loading shows incremental UI updates

**Error Handling (`/error-handling/`)**: Comprehensive error management patterns:
- Global and route-level `error.tsx` boundaries with retry functionality
- Client-side error reporting and user feedback systems
- Network, server, and validation error examples with fallback UI

**Performance Monitoring (`/performance/`)**: Web Vitals and metrics tracking:
- Mock performance data generation with realistic patterns
- Core Web Vitals visualization with charts and metrics cards
- Client-side monitoring components for real performance tracking

**API Routes with Background Tasks (`/api-routes/`)**: Next.js `after` API demo:
- `/api/contact-with-after/route.ts` shows immediate response + background processing
- Form submissions trigger background tasks without blocking user experience

### Next.js 16 Compatibility Requirements

**Critical**: All dynamic routes must properly handle async `params`:

```typescript
// ✅ Correct (Next.js 16+)
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  // Use id here
}

// ❌ Incorrect (will cause errors)
export default async function Page({
  params,
}: {
  params: { id: string };
}) {
  const id = params.id; // Error: params must be awaited
}
```

### Data Fetching Conventions

Always handle API failures gracefully with fallback data:

```typescript
try {
  const res = await fetch(url, { next: { revalidate: 60 } });
  data = await res.json();
} catch (e) {
  // Fallback data for demo reliability
  data = { /* static fallback */ };
}
```

## Development Workflow

### Essential Commands

- `pnpm dev` - Development with **Turbopack** (faster than Webpack)
- `pnpm dev:safe` - Clean development start (removes `.next/` cache)  
- `pnpm lint` - **Biome** linting (replaces ESLint + Prettier)
- `pnpm format` - Biome formatting
- `pnpm type-check` - TypeScript validation without build

### Technology Stack Specifics

- **Biome** replaces ESLint/Prettier - use `biome.json` for configuration
- **Tailwind CSS v4** - No config file needed, uses PostCSS only  
- **Turbopack** - Default bundler for dev mode (not Webpack)
- **pnpm** - Package manager (not npm/yarn)
- **Lefthook** - Git hooks management via `lefthook.yml`
- **TypeScript 5.9** - Strict mode with Next.js typed routes enabled

### Commit Message Standards

**All commits must follow Conventional Commits format** with optional Gitmojis as a senior developer would write:

**Format**: `[emoji] <type>(<scope>): <description>` or `<type>(<scope>): <emoji> <description>`

**Common Types**:
- `feat`: ✨ New features or enhancements
- `fix`: 🐛 Bug fixes  
- `chore`: 🔧 Maintenance tasks (deps, tooling, etc.)
- `build`: 📦 Build system or external dependencies
- `docs`: 📝 Documentation changes
- `refactor`: ♻️ Code restructuring without functionality changes
- `style`: 💄 Code formatting, no logic changes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jalezi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
