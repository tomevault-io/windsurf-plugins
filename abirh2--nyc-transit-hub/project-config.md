---
trigger: always_on
description: **Project:** NYC Transit Hub
---

# NYC Transit Hub - Cursor Rules

## Project Overview

**Project:** NYC Transit Hub  
**Purpose:** Personal web app providing real-time MTA transit info, analytics, and accessibility-aware routing  
**Stack:** Next.js (App Router), TypeScript, HeroUI, Tailwind CSS, Prisma/Drizzle, PostgreSQL (Supabase), Recharts

### Key Data Sources
- MTA GTFS-Realtime feeds (subway, bus, LIRR, Metro-North)
- Service alerts (JSON/GTFS)
- Elevator/escalator status feeds
- Static GTFS for station metadata

### Core Features
- Real-time train tracker and station boards
- Accessibility-aware routing (elevator/escalator status)
- Reliability and disruption analytics
- Crowding estimation
- Personal commute assistant

### Node.js Version Requirement
This project requires Node.js >= 20.9.0. The terminal may reset to an older version between commands.

**Always prefix Node.js commands with the correct nvm version:**
```bash
nvm use 20.19.5 && npm run dev
nvm use 20.19.5 && npm run build
nvm use 20.19.5 && npm install <package>
```

---

## Collaboration & Flexibility

### Plans Are Guidelines, Not Constraints

Documentation and plans (including `planForApp.md`) are starting points, not rigid requirements:
- Deviate from the plan when it improves code quality or app functionality
- Prioritize best practices over strict adherence to initial designs
- Refactor structure if a better approach becomes apparent during implementation
- The goal is a well-built app, not perfect plan compliance

### Ask Questions, Don't Assume

Assumptions lead to rework. When in doubt:
- Ask clarifying questions before implementing
- Propose alternatives when you see a better approach
- Flag potential issues early rather than discovering them later
- Provide feedback on trade-offs (performance, complexity, maintainability)

The more questions asked upfront, the fewer problems downstream. Never hesitate to challenge an approach if there's a better way.

---

## Code Quality Standards

### Professional Engineering Style

Write code as a senior software engineer would. All code should be production-quality, maintainable, and self-documenting.

**Modularity:**
- Keep functions short and focused (single responsibility)
- Extract logic into well-named helper functions
- Avoid deep nesting (max 2-3 levels); use early returns and guard clauses
- Split large components into smaller, composable pieces

**Naming Conventions:**
- Use descriptive, meaningful names that reveal intent
- Variables: `stationDepartures`, `activeAlerts`, `elevatorStatus`
- Functions: `fetchRealtimeArrivals()`, `parseGtfsResponse()`, `calculateOnTimeRate()`
- Components: `StationBoard`, `AlertCard`, `ReliabilityChart`
- Avoid abbreviations unless universally understood (e.g., `id`, `url`)

**Code Efficiency:**
- Prefer concise solutions over verbose ones
- If something can be done in 5 lines instead of 50, use 5 lines
- Use modern JavaScript/TypeScript features (destructuring, optional chaining, nullish coalescing)
- Avoid unnecessary abstractions; add complexity only when it provides clear value

**TypeScript Standards:**
- Enable strict mode; never use `any`
- Create explicit interfaces for all data structures
- Handle null/undefined explicitly
- For external data: validate with Zod, then transform and assert types

---

## Framework & Library Guidelines

### Version Policy

Always use the latest stable versions of all libraries and frameworks. Do not pin to older versions unless there is a documented incompatibility. Keep dependencies updated.

### Next.js (App Router)

- Use Server Components by default; add `'use client'` only when needed
- Leverage server-side data fetching in page components
- Use Route Handlers for API endpoints (`/app/api/...`)
- Follow the App Router file conventions (`page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`)
- Use `next/image` for optimized images
- Use `next/link` for client-side navigation

### HeroUI Components

- Use HeroUI components as the primary UI library
- Follow HeroUI patterns for forms, modals, cards, and navigation
- Extend with Tailwind utilities when needed
- Keep component props explicit and typed

### Tailwind CSS

- Use Tailwind utility classes for styling
- Extract repeated patterns into components, not custom CSS
- Use CSS variables for theme consistency
- Follow mobile-first responsive design

### Database & ORM (Prisma/Drizzle)

- Define clear, normalized schemas
- Use migrations for all schema changes
- Write type-safe queries
- Handle database errors gracefully
- Use transactions for multi-step operations

### Data Fetching

- Use TanStack Query or native fetch with proper caching
- Implement proper loading and error states
- Set appropriate cache/revalidation strategies
- Handle stale data gracefully
- **Deduplicate before rendering:** APIs often return per-relation data (e.g., one arrival per station per train). Always deduplicate by primary identifier (e.g., `tripId`) before rendering lists to avoid duplicate keys and inflated counts.

---

## Source of Truth

**All documentation might be outdated.** The only reliable sources of truth are:
1. **Actual codebase** - Code as it exists now
2. **Live configuration** - Environment variables, configs as actually set
3. **Running infrastructure** - How services actually behave

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abirh2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
