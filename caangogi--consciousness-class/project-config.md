---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Consciousness Class — a Next.js 15 (App Router) platform for holistic creators / therapists / coaches. The product is a unified "Asset Catalog" with six sellable asset types: **Course, Membership, Coaching, Podcast, Community, Download**. Users have one of four roles: `student`, `creator`, `admin`, `superadmin`.

The active roadmap and architectural state-of-affairs lives in [docs/hoja-de-ruta.md](docs/hoja-de-ruta.md). Brand/style intent (Natureza palette, Apple HIG, `ios-list` patterns) is in [docs/blueprint.md](docs/blueprint.md). Read both before making product-shaped decisions.

## Commands

```bash
npm run dev          # next dev --turbopack on port 9003
npm run build        # next build
npm run start        # next start
npm run lint         # next lint
npm run typecheck    # tsc --noEmit (the source of truth — see "Build vs typecheck" below)

npm test             # vitest run (unit + integration, single pass)
npm run test:watch   # vitest --watch (development)
npm run test:ui      # vitest --ui (visual debug in browser)
npm run test:e2e     # playwright test (auto-starts npm run dev on :9003)
npm run test:e2e:ui  # playwright UI mode for debugging
npm run test:e2e:headed  # see the browser while tests run

npm run genkit:dev   # Genkit dev server, entry src/ai/dev.ts
npm run genkit:watch # Genkit dev server with --watch
```

Utility scripts (run with `npx tsx` or `node`):
- [scripts/seed-db.ts](scripts/seed-db.ts) — seeds Firestore (loads `.env.local` then `.env`).
- [scripts/set-superadmin.js](scripts/set-superadmin.js) — promotes a user to `superadmin` in Firestore.

## Testing strategy (READ BEFORE CODING)

Full philosophy in [documentation/testing-strategy.md](documentation/testing-strategy.md). Three modes by domain:

| Mode | Where it applies |
|------|-----------------|
| **TDD strict** (test-first, separate `test(...)` commit before `feat(...)`) | `src/backend/payments/`, `src/backend/wallet/`, `src/backend/referrals/`, `src/backend/booking/` (state machine + refund rules), `src/app/api/webhooks/` |
| **Test-after rigorous** (test before merge, ≥70% module coverage) | API routes (wire-up), services, helpers, utilities |
| **Eval-first** (golden set + human review, no `expect()` against LLM output) | Everything in `src/ai/` and AI endpoints |

**Critical rule for TDD-strict directories:** the PR must show the failing `test(...)` commit *before* the `feat(...)` commit that makes it pass. If review can't see that sequence in git history, the PR is rejected.

When you (Claude Code) are about to edit a file in a TDD-strict directory, **stop and write the failing test first**. Commit it. Then write the implementation. Commit that separately.

### E2E (Playwright)

E2E tests live in `e2e/` (NOT mixed with unit tests in `src/`). Config in [playwright.config.ts](playwright.config.ts) auto-starts `npm run dev` on port 9003. First-time setup on a new machine: `npx playwright install chromium`.

The current smoke test ([e2e/smoke.spec.ts](e2e/smoke.spec.ts)) verifies that the public surfaces (home, /products, /courses redirect) render without 5xx. Behavioral E2E (full checkout, booking, RAG companion) lands per-feature in fases 5/6 once seedable Firebase + Stripe test data exists.

## Build vs typecheck

`next.config.ts` sets `typescript.ignoreBuildErrors: true` and `eslint.ignoreDuringBuilds: true`. **`npm run build` will pass even with type errors.** Use `npm run typecheck` to verify TS correctness — never trust a green `build` alone. ([ts_errors.txt](ts_errors.txt) at the repo root is a historical dump from one such run, not a live artifact.)

## Architecture

### Backend: hexagonal per asset type

Each business domain under [src/backend/](src/backend/) follows the same three-layer structure:

```
src/backend/<domain>/
  domain/          entities/, repositories/      (pure interfaces + entity classes)
  application/     <name>.service.ts             (use cases — orchestrates repos + Stripe + catalog)
  infrastructure/  dto/, repositories/           (Firebase repo impls, request/response DTOs)
```

Domains: `course`, `membership`, `coaching`, `community`, `podcast`, `download`, `booking`, `catalog`, `enrollment`, `progress`, `referral`/`referrals`, `finance`, `payments`, `user`, `shared`. Wiring goes **API route → Service → Repository interface → Firebase impl**; never have a route talk to Firestore directly.

### Catalog unification

Whenever a Course/Membership/Coaching/Podcast/Community/Download asset is created or updated, its service must sync a lightweight `CatalogItem` financial record via `CatalogService` (see the pattern in [src/backend/course/application/course.service.ts](src/backend/course/application/course.service.ts)). The unified storefront and creator catalog (`/api/creator/catalog`, `/dashboard/products`) read from this catalog, not from per-domain collections. When adding a new asset type, replicate the catalog sync — features that skip it become invisible to checkout.

### Enrollments


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caangogi/consciousness-class](https://github.com/caangogi/consciousness-class) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-24 -->
