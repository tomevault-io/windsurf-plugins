---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Setup
```bash
pnpm install           # Install dependencies
pnpm prepare          # Setup git hooks (husky + lint-staged)
cp .env.example .env  # Copy environment variables
```

### Development Server
```bash
pnpm dev              # Start Next.js development server with Turbopack (default)
pnpm dev:webpack      # Start with webpack instead of Turbopack
```

### Building
```bash
pnpm build            # Production build with Turbopack (default)
pnpm build:webpack    # Production build with webpack
pnpm start            # Start production server
```

### Code Quality
```bash
pnpm lint             # Run ESLint
pnpm lint:fix         # Fix ESLint issues automatically
pnpm prettier         # Format all files with Prettier
pnpm typegen          # Generate Next.js typed routes and run TypeScript compiler
```

### Testing
```bash
pnpm test             # Run Vitest tests once
pnpm test:watch       # Run Vitest in watch mode
```

### Internationalization (Tolgee)
```bash
pnpm tolgee:pull      # Pull translations from Tolgee
pnpm tolgee:push      # Push translations to Tolgee
pnpm tolgee:sync      # Sync translations bidirectionally
```

## Architecture Overview

### Tech Stack
- **Next.js 15** (App Router) with Turbopack
- **React 19** with React Compiler RC
- **TypeScript** with strict configuration
- **Tailwind CSS** for styling
- **Tolgee** for i18n (English & Polish)
- **Vitest** for testing
- **Zod** for runtime validation

### Project Structure

```
src/
├── app/                       # Next.js App Router pages
│   ├── (pages)/              # Route group for main pages
│   ├── (cities)/             # Route group for city-specific pages
│   ├── layout.tsx            # Root layout with Tolgee provider
│   └── page.tsx              # Homepage
├── components/               # React components
│   ├── Navigation/           # Navigation components
│   └── ui/                   # shadcn/ui components
├── content/                  # Content configuration files
│   ├── cities.tsx           # City definitions and map positions
│   ├── events-discounts.ts  # Conference/event discount data
│   ├── software-discounts.ts # Software tool discount data
│   ├── learning-discounts.ts # Learning platform discount data
│   ├── communityParticipation.ts # Community surveys/initiatives
│   ├── additionalEvents.ts  # Static event data
│   └── partners.tsx         # Partner/sponsor data
├── tolgee/                   # Tolgee i18n configuration
│   ├── shared.ts            # Base Tolgee config
│   ├── server.tsx           # Server-side instance
│   ├── client.tsx           # Client-side provider
│   └── language.ts          # Language management
├── types/                    # TypeScript type definitions
├── utils/                    # Utility functions
├── hooks/                    # Custom React hooks
└── env.ts                    # Environment variable validation (T3 Env)

messages/
├── en.json                   # English translations
└── pl.json                   # Polish translations
```

### Key Architectural Patterns

#### Next.js App Router Structure
- **Route Groups**: `(pages)` and `(cities)` group routes without affecting URLs
- **Server Components by default**: Most components are Server Components for better performance
- **Client Components**: Marked with `'use client'` directive (e.g., interactive forms, animations)
- **Typed Routes**: Enabled via `typedRoutes: true` in next.config.ts

#### Internationalization with Tolgee
- **Server Components**: Use `await getTranslate()` from `@/tolgee/server`
- **Client Components**: Use `useTranslate()` hook from `@tolgee/react`
- **Translation keys**: Organized by section (e.g., `navigation.home`, `hero.title`)
- **Static data fallback**: Translation files in `messages/` for development without API key
- **In-context editing**: Hold Alt + click on text to edit translations

#### Content Management
All promotional content, discounts, and community initiatives are configured via TypeScript files in `src/content/`:
- Discount banners automatically hide after expiry date
- Type-safe with Zod schemas
- Support for gradients, icons, and metadata

#### Events Data Flow
1. Events fetched from external API (configured via `EVENTS_API_URL`)
2. Validated with Zod schema (`EventsSchema`)
3. Cached for 1 hour (`revalidate: 3600`)
4. Merged with static additional events
5. Rendered in `EventCard` components

#### Environment Variables
- Validated using `@t3-oss/env-nextjs` in `src/env.ts`
- Runtime validation with Zod schemas
- Server-only variables: `EVENTS_API_URL`, `SITE_URL`, `DISCORD_SERVER_ID`
- Public variables: `NEXT_PUBLIC_TOLGEE_API_KEY`, `NEXT_PUBLIC_TOLGEE_API_URL`

#### Styling Approach
- Tailwind CSS with custom configuration
- shadcn/ui components in `src/components/ui/`
- Utility function `cn()` for conditional class merging (`clsx` + `tailwind-merge`)
- Safelist for dynamic gradient classes (see `tailwind.config.ts`)

#### Pre-commit Hooks
- Husky runs lint-staged on commit
- Prettier formats staged files automatically
- Configured in `.lintstagedrc`

## Important Development Notes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [naugtur/meetjs.pl](https://github.com/naugtur/meetjs.pl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
