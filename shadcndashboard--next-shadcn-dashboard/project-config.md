---
trigger: always_on
description: This file provides essential information for AI coding agents working on this project. It contains project-specific details, conventions, and guidelines that complement the README and CLAUDE.md.
---

# AGENTS.md - AI Coding Agent Reference

This file provides essential information for AI coding agents working on this project. It contains project-specific details, conventions, and guidelines that complement the README and CLAUDE.md.

---

## Project Overview

**Shadcn Dashboard** is a Next.js admin dashboard built with:

- **Framework**: Next.js 16.3 (App Router), React 19
- **Language**: TypeScript 7, type-checked via CLI as part of `next build`
- **Instant Navigations**: `cacheComponents` + `partialPrefetching` in `next.config.ts`
- **Routing**: File-based App Router — `app/auth/` (standalone auth pages) and `app/(dashboard-layout)/` (sidebar + header shell)
- **Styling**: Tailwind CSS v4
- **UI Components**: Shadcn UI-style primitives on Base UI (`@base-ui/react`)
- **Forms**: plain `useState` + shadcn UI primitives, following `app/(dashboard-layout)/pages/form/page.tsx`
- **Data fetching**: `swr` calling real Next.js Route Handlers under `app/api/**/route.ts`
- **Charts**: `recharts` (see below)
- **Icons**: `lucide-react` is the configured default (`components.json` → `iconLibrary: "lucide"`); `@iconify/react` also appears in some components
- **Rich text**: `@tiptap/*`, used in the blog post editor (`app/components/shared/editor/tiptap-edit.tsx`)
- **Package Manager**: pnpm (`pnpm-lock.yaml` is the lockfile of record)

This repo has a real backend layer: `app/api/**/route.ts` Route Handlers, not a mocked SPA.

---

## Project Structure

```
/app
├── layout.tsx                  # Root layout, fonts, metadata, providers
├── globals.css                 # Tailwind v4 + theme tokens
├── not-found.tsx                # Custom 404
├── api/                          # Route Handlers (real backend)
│   ├── global-fetcher.ts          # SWR fetcher functions (GET/POST/PUT)
│   └── blog/, notes/, ticket/       # Per-feature route handlers
├── auth/                          # Auth routes, outside the dashboard shell
│   ├── auth2/                       # Two-factor auth pages
│   ├── authforms/                    # Login, register, forgot-password, etc.
│   ├── error/, maintenance/
├── (dashboard-layout)/              # Dashboard shell (sidebar + header)
│   ├── layout.tsx, page.tsx
│   ├── apps/                          # Blog, notes, tickets pages
│   ├── pages/                          # form, tables pages
│   ├── layout/                          # header/sidebar/footer components
│   ├── icons/, types/
├── components/                        # App-scoped components
│   ├── dashboards/                      # Dashboard widgets/charts
│   ├── apps/                             # Feature components (blog/notes/tickets)
│   ├── tables/                            # TanStack Table wrappers (DataTable, CheckboxTable, etc.)
│   ├── animated-components/                # Framer Motion / dropzone components
│   ├── shared/, icons/
├── context/                            # React context providers, one per domain
│   └── blog-context/, notes-context/, ticket-context/
└── css/                                  # Page/style-specific CSS

/components
├── ui/                                    # Shadcn-style primitives (Radix/Base UI wrapped with cva + cn())
└── Themeprovider.tsx                        # Theme (dark/light) provider

/hooks                                        # Custom hooks (e.g. use-mobile.ts)
/lib
└── utils.ts                                   # cn() and shared helpers
```

---

## Build & Development Commands

```bash
# Install dependencies
pnpm install

# Development server (http://localhost:3000)
pnpm dev

# Type-check + production build
pnpm build

# Serve the production build
pnpm start

# Lint
pnpm lint
```

`pnpm build` runs Next's build-time type-check — TypeScript errors fail the build, not just lint.

---

## Routing Pattern

Standard Next.js App Router file-based routing:

1. Dashboard pages live under `app/(dashboard-layout)/`, wrapped by that group's `layout.tsx` (sidebar + header shell).
2. Standalone pages (auth, error, maintenance) live under `app/auth/`, outside the dashboard shell.
3. Adding a new dashboard page = adding a `page.tsx` under the appropriate `app/(dashboard-layout)/...` folder — no manual route registration needed (unlike a router-config SPA).
4. If the page needs a sidebar entry, wire it into `app/(dashboard-layout)/layout/vertical/sidebar/sidebaritems.ts`.

---

## Data & Fetching Pattern

This project has a real backend layer:

1. Route Handlers live in `app/api/<feature>/route.ts` (and feature-specific data/helper files alongside, e.g. `app/api/blog/route.ts`).
2. Client components/contexts fetch via `swr` using the shared fetchers in `app/api/global-fetcher.ts`.
3. Feature state (blog posts, notes, tickets) is exposed through a dedicated context in `app/context/<feature>-context/`, wrapping the SWR call and exposing state + setters.

When adding a new feature that needs data, mirror the blog/notes/ticket pattern: route handler → context provider → view/components consuming the context.

---

## Component & Styling Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shadcndashboard/next-shadcn-dashboard](https://github.com/shadcndashboard/next-shadcn-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
