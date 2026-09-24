---
trigger: always_on
description: **Next.js 16 + React 19 + TypeScript 5.9 + @flamingo-stack/openframe-frontend-core (0.0.653)**
---

# OpenFrame Frontend - Claude Development Guide

**Next.js 16 + React 19 + TypeScript 5.9 + @flamingo-stack/openframe-frontend-core (0.0.653)**

> Comprehensive instructions for Claude when working with the OpenFrame Frontend service.

## Core Principles

**MANDATORY REQUIREMENTS:**
1. ALL UI components MUST use `@flamingo-stack/openframe-frontend-core` — never create custom UI primitives
2. ALL styling MUST use ODS design tokens (no hardcoded colors)
3. Follow WCAG 2.1 AA accessibility standards
4. Use `react-hook-form` + `zod` for forms; use `useToast` for all API feedback
5. Use `react-relay` for GraphQL data fetching wherever possible — the codebase is gradually migrating to Relay. Use `@tanstack/react-query` for REST APIs and for legacy GraphQL code that has not been migrated yet. Do NOT introduce new raw-POST GraphQL calls.

## Setup & Commands

### Quick Setup
```bash
npm install
cp .env.local.example .env.local   # or create manually:
echo "NEXT_PUBLIC_TENANT_HOST_URL=http://localhost" >> .env.local
echo "NEXT_PUBLIC_APP_MODE=oss-tenant" >> .env.local
npm run dev
```
Access: http://localhost:3000

### All Commands
| Command | Purpose |
|---------|----------|
| `npm run dev` | Dev server (port 3000, `PORT` env to override) |
| `npm run build` | Production build (`generate-enums` + `relay-compiler` + `next build`; standalone output in `dist/`) |
| `npm run build:export` | Static-export build (`OPENFRAME_BUILD_TARGET=export`) — SPA bundle for Capacitor/Tauri native shells |
| `npm run build:local` | Production build with webpack |
| `npm run start` | Start production server |
| `npm run start:standalone` | Serve the standalone build (`dist/standalone/server.js`) |
| `npm run type-check` | TypeScript validation (`tsc --noEmit`) |
| `npm run relay` | Relay compiler — regenerates `src/__generated__/` artifacts |
| `npm run relay:watch` | Relay compiler in watch mode |
| `npm run fetch-schema` | Pull `schema.graphql` from a backend via introspection (`-- --endpoint <url> --token <JWT>`) |
| `npm run generate-enums` | Regenerate `src/generated/schema-enums.ts` (enum const+type) from `schema.graphql` |
| `npm run lint` | ESLint — the fast pass (`eslint.config.mjs`), cached |
| `npm run lint:ci` | What CI blocks on: the fast pass minus the `relay/unused-fields` backlog (`eslint.ci.mjs`) |
| `npm run lint:fix` | ESLint autofix (import order, unused imports, type imports) |
| `npm run lint:types` | ESLint type-aware pass (`eslint.types.mjs`; slow, needs an 8 GB heap) |
| `npm run lint:cycles` | ESLint `import/no-cycle` pass (`eslint.cycles.mjs`; walks the whole import graph) |
| `npm run format` | Prettier check |
| `npm run format:fix` | Prettier write |
| `npm run core:link` / `core:unlink` | yalc-link/unlink the core library for local lib development |

### Pre-commit Hooks
Husky (`.husky/pre-commit`) is **staged-file-scoped**: it runs ESLint (via `eslint.ci.mjs`, the same set CI blocks on) and `prettier --check` on the staged frontend files, plus `tsc --noEmit` with errors filtered to staged files only. A clean commit does not require the whole repo to pass, but today it very nearly does: `npm run lint:ci` — the fast pass minus the `relay/unused-fields` backlog — is green, and CI blocks on it. Keep `npm run type-check` and `npm run format` green too.

### Environment Variables

**Required:**
```bash
NEXT_PUBLIC_TENANT_HOST_URL=http://localhost   # Backend API host
NEXT_PUBLIC_APP_MODE=oss-tenant                # App mode (see below)
```

**SaaS deployment:**
```bash
NEXT_PUBLIC_SHARED_HOST_URL=https://auth.openframe.ai   # Shared auth host
NEXT_PUBLIC_GTM_CONTAINER_ID=GTM-XXXXXXX                # Google Tag Manager
```

**Dev auth:**
```bash
NEXT_PUBLIC_ENABLE_DEV_TICKET_OBSERVER=true   # Dev ticket auth mode (Bearer tokens instead of cookies)
```

Feature flags are **not** env vars — they are server-loaded via GraphQL (see Feature Flags below). Native-shell env split is documented in `.env.export.example`.

### Payment UI Visibility (native app builds)

`src/lib/billing-visibility.ts` is the single switch for every payment surface, in three tiers keyed
on the shell: **web** shows and changes everything; **desktop** (`isBillingReadOnly()` =
`isDesktopShell()`) shows everything and changes nothing; **mobile** (`isBillingHidden()` =
`isMobileShell()`) shows none of it. Billing runs through Stripe on the web; App Store Guideline 3.1.1
forbids showing plans, prices, invoices, or any CTA leading to a non-IAP purchase, and Google Play
treats an in-app Stripe Checkout for a digital subscription as bypassing Play Billing. Desktop ships
outside any store, so neither rule reaches it — but payment still belongs in the browser (bank
verification steps, autofill, saved cards), hence read-only plus one exit.

No env var backs this: the shell injects `window.Capacitor` / the Tauri globals itself, so a native
build can't forget to declare what it is, and the web bundle can't be misconfigured into hiding its
billing.

### Which shell am I in? (`src/lib/platform.ts`)

The same export runs in three places, so ask the axis that owns the feature — never "is this native?":

- `isAppShell()` — either shell. Shell-custodied tokens, no Next server behind the origin (so

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flamingo-stack/openframe-oss-frontend](https://github.com/flamingo-stack/openframe-oss-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
