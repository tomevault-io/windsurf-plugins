---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

NextMedal is a Next.js 16 + Sanity CMS website template built by Medal Social. It features Server Components, Turbopack, i18n support (Norwegian/English/Arabic), Docker-optimized standalone output, and a flexible article system for content publishing.

## Commands

```bash
# Development
pnpm dev                    # Start dev server with Turbopack (http://localhost:3000)
pnpm build                  # Production build
pnpm start                  # Run production build

# Code Quality
pnpm lint                   # Run Biome linting
pnpm format                 # Auto-format with Biome
pnpm typecheck              # TypeScript type checking

# Testing (Vitest)
pnpm test                   # Run all unit/integration/component tests
pnpm test:watch             # Run tests in watch mode
pnpm test:unit              # Run unit tests only
pnpm test:components        # Run component tests only
pnpm test:integration       # Run integration tests only
pnpm test:contracts         # Run API contract tests
pnpm test:coverage          # Run tests with coverage report

# E2E Testing (Playwright)
pnpm e2e                    # Run full E2E tests
pnpm e2e:smoke              # Run smoke tests (quick critical paths)
pnpm e2e:visual             # Run visual regression tests
pnpm e2e:visual:update      # Update visual baselines
pnpm e2e:a11y               # Run accessibility tests
pnpm e2e:perf               # Run performance/Lighthouse tests

# Docker
pnpm docker:build           # Build production Docker image
```

## Architecture

### Directory Structure

```
project-root/
├── src/                    # Production source code
│   ├── app/                # Next.js 16 App Router
│   │   ├── (frontend)/     # Main website routes with [locale] parameter
│   │   ├── (studio)/       # Sanity CMS Studio at /studio
│   │   └── api/            # API routes (search, draft-mode)
│   ├── components/         # React components
│   │   ├── ui/             # Reusable base UI primitives (41 components)
│   │   ├── blocks/         # Content modules and page-level components (130+ files)
│   │   │   ├── modules/    # Content modules (hero, features, testimonials, etc.)
│   │   │   ├── objects/    # Reusable objects (CTA, icons, video, etc.)
│   │   │   ├── layout/     # Layout components (header, footer, banner)
│   │   │   └── seo/        # SEO components (JSON-LD, breadcrumbs)
│   │   ├── layout/         # Additional layout utilities
│   │   ├── icons/          # Icon components
│   │   └── dashboard/      # Sanity Studio dashboard components
│   ├── sanity/schemaTypes/ # 66 Sanity schema definitions
│   ├── lib/                # Core utilities (logger, env, utils, safe-action)
│   └── i18n/               # Internationalization config
│
└── tests/                  # All tests (85+ test files)
    ├── unit/               # Pure function tests (Vitest, ~30 files)
    │   ├── lib/            # Utils, helpers, pure logic
    │   ├── sanity/         # Sanity utilities
    │   └── config/         # Configuration validation
    ├── components/         # React component tests (Vitest + Testing Library, ~41 files)
    │   ├── ui/             # Base UI primitives
    │   ├── blocks/         # Block components
    │   └── layout/         # Layout components
    ├── integration/        # Multi-module tests (Vitest, ~14 files)
    │   ├── api/            # API route integration
    │   ├── hooks/          # Custom hooks
    │   └── forms/          # Form validation
    ├── contracts/          # API contract tests (Vitest + Zod)
    ├── e2e/                # End-to-end tests (Playwright, ~13 files)
    │   ├── smoke/          # Quick critical path tests
    │   ├── specs/          # Full E2E test specs
    │   ├── visual/         # Visual regression tests
    │   ├── performance/    # Lighthouse performance tests
    │   └── accessibility/  # WCAG compliance tests
    ├── fixtures/           # Shared test data
    │   ├── sanity/         # Sanity mock data
    │   └── playwright/     # Playwright fixtures
    └── setup/              # Test configuration
```

### Key Patterns

- **Server Components by default**: Use `'use client'` only when needed
- **Sanity integration**: Schemas in `sanity/schemaTypes/`, Studio at `/studio`
- **i18n routing**: `[locale]` dynamic segment for Norwegian (nb), English (en), and Arabic (ar)
- **Environment validation**: Zod-validated env vars in `lib/env.ts`
- **Structured logging**: Pino logger in `lib/logger.ts`
- **Safe server actions**: Use `next-safe-action` wrapper in `lib/safe-action.ts`
- **Article system**: Flexible content publishing with categories, authors, and SEO optimization

## Internationalization (i18n)

### Single Source of Truth

**All language configuration is centralized in `src/i18n/config.ts`**. This is the ONLY file you need to edit to add a new language.

```typescript
// src/i18n/config.ts
export const LOCALE_CONFIG = {
  en: { title: 'English', dateLocale: enUS, dir: 'ltr' },
  nb: { title: 'Norsk', dateLocale: nb, dir: 'ltr' },
  ar: { title: 'العربية', dateLocale: ar, dir: 'rtl' },
} as const;
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Medal-Social/NextMedal](https://github.com/Medal-Social/NextMedal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
