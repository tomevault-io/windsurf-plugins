---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Cross-site standards**: See `STANDARDS.md` in this repo for shared conventions
> that apply across all TK web properties. This file documents SchedulEd-specific
> details only. STANDARDS.md takes precedence on any conflicting guidance.

## Identity

**SchedulEd** (repo: AIStudyPlans) is the AI-powered study plan generator web application in the Herculean ecosystem. It is a consumer-facing Next.js 15 landing page and waitlist funnel, not an infrastructure agent — it serves end-users (students, parents, educators) rather than monitoring or managing other systems.

**Owner:** TK @ Bridging Trust AI
**Stack:** Next.js 15, React 19, TypeScript, Tailwind CSS v3, Supabase, Resend, Auth.js v5
**Deploy:** Azure Static Web Apps

## Scope

| Domain | Description |
|--------|-------------|
| Landing page | Hero, features, pricing, FAQ, waitlist form |
| Waitlist funnel | Supabase storage + Resend confirmation emails |
| Contact forms | Sales and support contact endpoints |
| Authentication | Auth.js v5 with Microsoft Entra ID |
| Admin dashboard | Email stats, CI status, feedback, monitoring |
| Feedback system | Campaign-based feedback collection |

## Standards

This repo follows **Herculean Ecosystem Standards v1.1** (enforced by `.github/workflows/standards-check.yml`):

- **Linter**: Biome (`biome.json`) for formatting + linting; ESLint (`eslint.config.mjs`) for TypeScript/Next.js-specific rules
- **Pre-commit**: `.pre-commit-config.yaml` with biome-check and no-dot-env hooks
- **Secrets**: 1Password via `.env.1p.template` with `op://` references; never commit `.env` files
- **Module system**: ESM (`"type": "module"` in `package.json`)
- **Node version**: Pinned in `.nvmrc`, enforced via `engines.node`
- **Dependencies**: Exact versions, shrinkwrap committed

## Project Overview

SchedulEd (AIStudyPlans) is a Next.js 15 landing page for an AI-powered study plan generator. It's deployed on Azure Static Web Apps and uses Supabase as the backend database, Resend for email delivery, and Auth.js v5 (next-auth) for authentication.

## Commands

### Development
```bash
npm run dev          # Start dev server on localhost:3000
npm run build        # Production build
npm run start        # Serve production build
```

### Testing
```bash
npm test                                # Run Vitest unit tests
npm test -- __tests__/Header.test.tsx   # Run a single test file
npm run test:watch                      # Vitest in watch mode
npm run test:coverage                   # Vitest with v8 coverage (70% threshold)
npm run test:e2e                        # Playwright E2E tests (auto-starts dev server)
npm run test:e2e:ui                     # Playwright with interactive UI
npm run test:all                        # lint + typecheck + unit + e2e
```

### Validation (run before pushing)
```bash
npm run validate         # Full: lint + typecheck + tests + build
npm run validate:quick   # Fast: lint + typecheck + build (no tests)
```

### Linting & Type Checking
```bash
npm run lint         # ESLint 9 flat config (eslint.config.mjs)
npm run lint:fix     # ESLint with auto-fix
npm run typecheck    # tsc --noEmit
```

**Lint gotchas**: `@typescript-eslint/no-unused-vars` is `"error"` — any unused import or variable fails lint. `@typescript-eslint/no-explicit-any` is `"warn"` — `any` usage produces warnings. `no-console` is `"warn"` — use `// eslint-disable-next-line no-console` for intentional console.log. The `--max-warnings 0` flag means any warning fails CI.

### Docker
```bash
./run-docker.sh start   # Start dev environment on localhost:3001
./run-docker.sh stop    # Stop dev environment
./run-docker.sh test    # Run unit tests in Docker
./run-docker.sh e2e     # Run E2E tests in Docker
```

### Dependencies
```bash
npm install package-name --save-exact   # Always use exact versions (no ^ or ~)
npm shrinkwrap                          # After adding deps, lock transitive deps
npm run validate:deps                   # Verify dependency standards
```

## Architecture

### Dual App Directory Structure
The project has **two** app entry points — this is important to understand:

- **`app/`** — Primary application directory. Contains all routes, API endpoints, and page-level components. This is where the main landing page (`app/page.tsx`) lives with Hero, Features, Pricing, FAQ, WaitlistForm, etc.
- **`src/app/`** — Secondary/legacy entry point with a separate `layout.tsx` and `page.tsx`. Contains a backup layout and minimal page.

The `app/` directory is the active one used by Next.js (it takes precedence).

### Component Organization
Components live in three locations:
- **`app/components/`** — Page-specific components used directly by routes (Header, Hero, Features, Footer, WaitlistForm, etc.)
- **`app/components/landing/`** — Alternative landing page section components (HeroSection, FeaturesSection, etc.)
- **`components/`** — Shared/reusable components (ErrorBoundary, auth Provider, admin components)

### Provider Hierarchy
`app/layout.tsx` wraps the app with: `ErrorBoundary` → `Providers` (AuthProvider + ThemeProvider) → `AnalyticsProvider`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [herculeanfit1/AIStudyPlans](https://github.com/herculeanfit1/AIStudyPlans) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
