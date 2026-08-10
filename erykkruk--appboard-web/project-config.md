---
trigger: always_on
description: AppBoard admin panel — an ASO (App Store Optimization) tool for managing apps, listings, screenshots, and metadata on the App Store and Google Play. Users arrive here from the public marketing website (appboard_website).
---

# AppBoard Admin Panel

## Overview

AppBoard admin panel — an ASO (App Store Optimization) tool for managing apps, listings, screenshots, and metadata on the App Store and Google Play. Users arrive here from the public marketing website (appboard_website).

---

## Tech Stack

| Category | Technology | Version |
|----------|-----------|---------|
| Framework | Next.js (App Router) | 16.x |
| UI Library | React | 19.x |
| Server State | TanStack React Query | 5.x |
| Auth | Better Auth (client) | 1.x |
| Styling | Tailwind CSS | 4.x |
| UI Components | shadcn/ui + Radix UI | latest |
| Drag & Drop | dnd-kit | latest |
| Icons | Lucide React | latest |
| Toasts | Sonner | latest |
| Theme | next-themes | latest |
| Linter | ESLint (next config) | 9.x |
| Testing | Bun test + RTL + happy-dom | latest |
| Package Manager | Bun | latest |

---

## Development Commands

| Command | Description |
|---------|------------|
| `bun dev` | Start dev server on port 6600 |
| `bun run build` | Production build |
| `bun run start` | Start production server |
| `bun run lint` | ESLint check |
| `bun test` | Run tests (Bun + happy-dom) |

---

## Directory Structure

```
src/
├── app/
│   ├── (app)/                   # Authenticated layout group
│   │   ├── layout.tsx           # Sidebar + providers
│   │   ├── page.tsx             # Dashboard redirect
│   │   ├── dashboard/           # Main dashboard
│   │   ├── apps/
│   │   │   └── [appId]/         # App detail pages
│   │   │       ├── layout.tsx   # App-level layout
│   │   │       ├── dashboard/   # App dashboard
│   │   │       ├── information/ # App metadata/listing
│   │   │       ├── versions/    # Version management
│   │   │       │   └── [versionId]/ # Version detail
│   │   │       │       ├── age-rating/
│   │   │       │       ├── graphics/
│   │   │       │       └── languages/
│   │   │       ├── publish/     # Publishing flow
│   │   │       ├── purchases/   # In-app purchases
│   │   │       ├── reviews/     # User reviews
│   │   │       ├── settings/    # App settings
│   │   │       └── setup/       # Initial setup
│   │   ├── onboarding/          # First-time setup
│   │   ├── settings/            # Workspace settings
│   │   └── templates/           # Listing templates
│   ├── (auth)/                  # Auth layout group (login/register)
│   ├── layout.tsx               # Root layout
│   ├── globals.css              # Global styles (Tailwind)
│   └── not-found.tsx            # 404 page
├── components/
│   ├── ui/                      # shadcn/ui components
│   ├── app-sidebar.tsx          # Main sidebar navigation
│   ├── page-header.tsx          # Page header component
│   ├── providers.tsx            # QueryClient + Theme providers
│   ├── actions-menu.tsx         # Context actions menu
│   ├── character-counter.tsx    # Input character counter
│   ├── prompt-editor.tsx        # AI prompt editor
│   ├── screenshot-crop-dialog.tsx
│   ├── screenshot-split-dialog.tsx
│   └── monetization-planner/    # Monetization planning widget
├── hooks/
│   ├── use-apps.ts              # App CRUD operations
│   ├── use-stores.ts            # Store connections
│   ├── use-settings.ts          # Workspace settings
│   ├── use-aso-profile.ts       # ASO profile management
│   ├── use-assets.ts            # Screenshots/graphics
│   ├── use-publishing.ts        # Publishing flow
│   ├── use-purchases.ts        # In-app purchases
│   ├── use-reviews.ts           # App reviews
│   ├── use-ai.ts                # AI features
│   ├── use-app-ai-prompts.ts    # AI prompt management
│   ├── use-age-rating.ts        # Age rating
│   ├── use-app-groups.ts        # App grouping
│   ├── use-auto-save.ts         # Auto-save hook
│   ├── use-capabilities.ts      # Feature capabilities
│   ├── use-mobile.ts            # Mobile detection
│   ├── use-monetization-chat.ts # Monetization AI chat
│   ├── use-privacy-declaration.ts # Privacy declarations
│   └── use-prompts.ts           # Prompt management
├── lib/
│   ├── api.ts                   # API client (fetch wrapper)
│   ├── auth-client.ts           # Better Auth client
│   ├── types.ts                 # Shared TypeScript types
│   ├── utils.ts                 # Utility functions (cn, etc.)
│   ├── aso-profile-csv.ts       # ASO profile CSV export
│   ├── listings-csv.ts          # Listings CSV export
│   ├── gp-data-safety-catalog.ts # Google Play data safety
│   └── privacy-catalog.ts       # Privacy catalog data
├── proxy.ts                     # API proxy configuration
└── test/                        # Test setup & utilities
```

---

## Architecture Pattern

**App Router + Hooks-based architecture** (no feature-based folders):

```
Pages (src/app/)  →  Hooks (src/hooks/)  →  API Client (src/lib/api.ts)  →  Backend
     ↓                     ↓
Components            TanStack Query
(src/components/)     (cache + mutations)
```

### Layer Responsibilities

| Layer | Location | Responsibility |
|-------|----------|----------------|
| **Pages** | `src/app/` | Route handling, layout, page composition |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erykkruk/appboard_web](https://github.com/erykkruk/appboard_web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
