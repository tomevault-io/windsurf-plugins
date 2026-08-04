---
trigger: always_on
description: This file provides essential information for AI coding agents working on this project. It contains project-specific details, conventions, and guidelines that complement the README and CLAUDE.md.
---

# AGENTS.md - AI Coding Agent Reference

This file provides essential information for AI coding agents working on this project. It contains project-specific details, conventions, and guidelines that complement the README and CLAUDE.md.

---

## Project Overview

**Shadcn Dashboard Free** is a client-side admin dashboard SPA built with:

- **Framework**: Vite 8 + React 19 (no server framework — pure client-side SPA)
- **Language**: TypeScript, compiled with `tsc` before every build
- **Routing**: `react-router` (client-side, `createBrowserRouter`)
- **Styling**: Tailwind CSS v4
- **UI Components**: shadcn-style primitives on Base UI (`@base-ui/react`)
- **Forms**: no form library currently wired up (`react-hook-form`/`zod`/`@hookform/resolvers` were removed as unused — reintroduce if a form feature actually needs them)
- **Data fetching / mocking**: `swr` for fetching, `msw` for mocked API responses
- **Charts**: `recharts` (the only charting library in the project)
- **Icons**: `lucide-react` and `@iconify/react` are the two icon packages in use
- **Deployment**: static `dist/` build served via Netlify redirect (`netlify.toml`) or Docker + nginx (`Dockerfile`, `nginx.conf`)
- **Package Manager**: npm (`package-lock.json` is the lockfile of record)

There is no backend in this repo. All "API" calls go through mock handlers (`msw`) backed by static data in `src/api/*/**-data.ts`.

---

## Project Structure

```
/src
├── App.tsx                 # Root app component
├── main.tsx                 # Vite entry point
├── routes/
│   └── Router.tsx           # All route definitions (react-router, createBrowserRouter), lazy-loaded via Loadable
├── layouts/
│   ├── full/                # Main dashboard shell (sidebar + header)
│   └── blank/                # Bare layout (auth pages, error pages)
├── views/                    # Page-level screens, one folder per route
│   ├── apps/                 # Blog, notes, tickets app screens
│   ├── auth/                  # Login/register/forgot-password/2FA/error/maintenance
│   ├── dashboards/            # Dashboard variants (e.g. modern)
│   ├── icons/                 # Icon showcase page
│   └── pages/                 # Tables, forms, user-profile, etc.
├── components/                # Reusable UI building blocks
│   ├── ui/                    # shadcn-style primitives (button, dialog, calendar, input-otp, chart, etc.)
│   ├── dashboards/             # Dashboard-specific widgets (e.g. total-sales chart)
│   ├── apps/                   # Feature components for blog/notes/tickets
│   ├── tables/                  # TanStack Table wrappers (DataTable, CheckboxTable, etc.)
│   ├── form/                    # Form field showcase/wrappers
│   ├── icons/                    # Icon registry/data (iconify-icons.tsx)
│   ├── animated-components/       # Motion/dropzone-based components
│   ├── user-profile/               # Profile page components
│   └── shared/                      # Cross-cutting shared components (ScrollToTop, StyleAwareWrapper, StyleDivider, dashboard-card)
├── context/                  # React context providers (one folder per domain)
│   ├── blog-context/, notes-context/, ticket-context
│   └── shadcntheme/           # Theme (dark/light) context
├── api/                       # Mock data + fetchers
│   ├── global-fetcher.ts       # SWR fetcher functions (GET/POST/PUT)
│   ├── blog/, notes/, ticket/    # Per-feature mock data (*-data.ts)
│   └── mocks/
│       ├── browser.ts            # MSW browser worker setup
│       └── handlers/mock-handlers.ts  # Combines all feature handlers
├── hooks/                     # Custom hooks (e.g. use-mobile.ts)
├── lib/
│   └── utils.ts                # cn() and shared helpers
├── types/                      # Shared TypeScript types (e.g. types/apps/*)
└── css/                          # Global styles

netlify.toml                  # SPA redirect (all paths → /index.html)
Dockerfile                    # Multi-stage build: npm ci → vite build → nginx
nginx.conf                    # SPA fallback for the Docker image
```

---

## Build & Development Commands

```bash
# Install dependencies
npm install

# Development server (http://localhost:5173)
npm run dev

# Type-check + production build
npm run build

# Preview the production build
npm run preview

# Lint (ESLint, must pass with zero warnings)
npm run lint
```

`npm run build` runs `tsc` before `vite build` — TypeScript errors fail the build, not just lint. Always run `npm run lint` before considering frontend changes done.

---

## Routing Pattern

All routes are declared in `src/routes/Router.tsx` using `react-router`'s `createBrowserRouter`. Every page component is:

1. Lazily imported with `lazy(() => import('../views/...'))`
2. Wrapped in `Loadable` (`src/layouts/full/shared/loadable/Loadable.tsx`) to provide a suspense fallback

When adding a new page:

1. Create the view under `src/views/<area>/<page>/index.tsx` (or similar), following sibling folders.
2. Add a `Loadable(lazy(() => import(...)))` declaration near related routes in `Router.tsx`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shadcndashboard/shadcndashboard](https://github.com/shadcndashboard/shadcndashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
