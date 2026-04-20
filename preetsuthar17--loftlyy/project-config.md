---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

- **Dev server**: `pnpm dev` (Next.js 16 with Turbopack)
- **Build**: `pnpm build`
- **Type check**: `pnpm typecheck`
- **Lint & format check**: `pnpm check` (runs Ultracite/Oxlint+Oxfmt)
- **Auto-fix lint & format**: `pnpm fix`
- **Optimize brand images**: `pnpm optimize` (runs `scripts/optimize-images.ts` via tsx)

No test runner is configured.

## Architecture

This is a **statically generated Next.js 16 app** (App Router, React 19) that serves as a brand identity reference — like Mobbin but for branding assets. It displays brand colors, typography, logos/assets, and usage guidelines.

### Data model

All brand data lives in `data/brands/` as TypeScript files exporting `Brand` objects (typed in `lib/types.ts`). `data/brands/index.ts` aggregates them into a sorted array and provides lookup helpers (`getAllBrands`, `getBrandBySlug`, `getBrandsByCategory`). To add a brand, create a new `.ts` file in `data/brands/`, export the `Brand` object, and import it in `data/brands/index.ts`. Brand images go in `public/brands/<slug>/`.

Categories are defined in `data/categories.ts`.

### Routing & i18n

Uses `next-intl` with 5 locales: en, es, fr, de, ja (always-prefixed URLs). Translation files are in `messages/<locale>.json`. The middleware (`middleware.ts`) handles locale detection/redirect. `i18n/routing.ts` defines locale config, `i18n/request.ts` loads messages.

Route structure:

- `app/[locale]/layout.tsx` — root locale layout (fonts, theme, i18n provider)
- `app/[locale]/(brands)/page.tsx` — brand listing / landing
- `app/[locale]/(brands)/[slug]/page.tsx` — individual brand detail (statically generated for all brand×locale combos)
- `app/[locale]/category/[category-slug]/page.tsx` — category view

### Key patterns

- **Static generation**: `generateStaticParams` is used for both locale and brand slug pages. All pages are pre-rendered.
- **Selective client hydration**: `NextIntlClientProvider` receives only `brand` and `nav` message namespaces (not the full message bundle).
- **Dynamic imports**: `BrandAssets`, `SimilarBrands`, and `BrandLegal` are lazy-loaded on brand detail pages.
- **Similar brands scoring**: `lib/filters.ts` contains a weighted scoring algorithm matching brands by shared categories, tags, color families, and typography styles.
- **Color classification**: `hexToColorFamily()` in `lib/filters.ts` converts hex colors to named families (red, blue, green, etc.) for filtering.

### UI

- Tailwind CSS v4 with `tw-animate-css`
- shadcn/ui components in `components/ui/`
- `class-variance-authority` + `tailwind-merge` (via `lib/utils.ts`) for component variants
- `@tabler/icons-react` for icons
- `next-themes` for dark mode
- **Always use `gap-*` instead of `space-x-*`/`space-y-*`** for spacing between children

## Code Quality

This project uses **Ultracite** (Oxlint + Oxfmt). Run `pnpm fix` before committing. Key rules:

- No `console.log`, `debugger`, or `alert` in production code
- Use `for...of` over `.forEach()` and indexed `for` loops
- Use Next.js `<Image>` component, not `<img>`
- React 19: use `ref` as a prop, not `React.forwardRef`
- Prefer Server Components; only use `"use client"` when necessary

# Ultracite Code Standards

This project uses **Ultracite**, a zero-config preset that enforces strict code quality standards through automated formatting and linting.

## Quick Reference

- **Format code**: `pnpm dlx ultracite fix`
- **Check for issues**: `pnpm dlx ultracite check`
- **Diagnose setup**: `pnpm dlx ultracite doctor`

Oxlint + Oxfmt (the underlying engine) provides robust linting and formatting. Most issues are automatically fixable.

---

## Core Principles

Write code that is **accessible, performant, type-safe, and maintainable**. Focus on clarity and explicit intent over brevity.

### Type Safety & Explicitness

- Use explicit types for function parameters and return values when they enhance clarity
- Prefer `unknown` over `any` when the type is genuinely unknown
- Use const assertions (`as const`) for immutable values and literal types
- Leverage TypeScript's type narrowing instead of type assertions
- Use meaningful variable names instead of magic numbers - extract constants with descriptive names

### Modern JavaScript/TypeScript

- Use arrow functions for callbacks and short functions
- Prefer `for...of` loops over `.forEach()` and indexed `for` loops
- Use optional chaining (`?.`) and nullish coalescing (`??`) for safer property access
- Prefer template literals over string concatenation
- Use destructuring for object and array assignments
- Use `const` by default, `let` only when reassignment is needed, never `var`

### Async & Promises

- Always `await` promises in async functions - don't forget to use the return value
- Use `async/await` syntax instead of promise chains for better readability
- Handle errors appropriately in async code with try-catch blocks
- Don't use async functions as Promise executors

### React & JSX

- Use function components over class components
- Call hooks at the top level only, never conditionally
- Specify all dependencies in hook dependency arrays correctly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [preetsuthar17/loftlyy](https://github.com/preetsuthar17/loftlyy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
