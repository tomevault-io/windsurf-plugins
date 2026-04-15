---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Russian Video & Text — a web app for watching Russian videos (ok.ru) and reading Russian texts (lib.ru) with synced transcripts, click-to-translate, and SRS flashcard review. Users paste URLs; the backend downloads, transcribes (Whisper), punctuates (GPT-4o), and chunks the content. Words highlight in sync with playback. Users build a flashcard deck by clicking words, which persists to Firestore via Google Sign-In auth.

## Commands

```bash
npm run dev           # Start both frontend and backend (kills stale servers first)
npm run dev:frontend  # Start Vite dev server only
npm run dev:backend   # Start Express backend only (uses --watch)
npm run build         # TypeScript check + production build
npm run lint          # Run ESLint
npm run test          # Run ALL tests (frontend typecheck + server unit + integration)
npm run server:install # Install backend dependencies
```

### Testing

```bash
# Run all tests (frontend + backend)
npm test

# Server unit tests only
cd server && npx vitest run

# Server tests in watch mode
cd server && npx vitest

# Run a single server test file
cd server && npx vitest run media.test.js
cd server && npx vitest run integration.test.js

# Run a single frontend test file
npx vitest run tests/landing-page.test.tsx
npx vitest run tests/app.test.tsx

# Run tests matching a pattern
cd server && npx vitest run -t "editDistance"
npx vitest run -t "renders feature sections"

# Integration tests against real APIs (requires network + API keys)
cd server && npm run test:integration
```

```bash
# Regenerate demo content (requires OPENAI_API_KEY, network access)
cd server && node scripts/generate-demo.js              # Both video + text
cd server && node scripts/generate-demo.js --video      # Video only
cd server && node scripts/generate-demo.js --text       # Text only
cd server && node scripts/generate-demo.js --upload-gcs # Also upload media to GCS
```

```bash
# E2E tests (Playwright — frontend only, all APIs mocked)
npm run test:e2e            # Run all E2E tests (headless)
npm run test:e2e:headed     # Run with visible browser
npm run test:e2e:ui         # Run with Playwright UI inspector

# Install Playwright browsers (first time only)
cd e2e && npx playwright install chromium
```

**Test files:**
- `tests/typecheck.test.js` — Runs `tsc -b` to catch TypeScript errors (30s timeout)
- `server/media.test.js` — Unit tests for heartbeat, stripPunctuation, editDistance, isFuzzyMatch
- `server/usage.test.js` — Unit tests for unified budget: cost tracking, requireBudget middleware, period resets, trackTranslateCost alias
- `server/usage-persistence.test.js` — Tests Firestore persistence: debounced writes, schema migration from legacy separate budgets
- `server/stripe.test.js` — Unit tests for Stripe subscription: status checks, middleware, checkout/portal sessions, webhook handling, cache invalidation
- `server/dictionary.test.js` — Unit tests for OpenRussian dictionary: convertStress, lookupWord (noun/verb/adjective/other), lemma fallback, graceful no-op
- `server/integration.test.js` — Mocks `media.js` + `dictionary.js`, tests all Express endpoints, SSE, session lifecycle, helmet headers, subscription endpoints
- `tests/deck-persistence.test.ts` — Unit tests for deck persistence: localStorage load/save, Firestore load/migration, debounced save factory, cleanup
- `tests/deck-enrichment.test.ts` — Unit tests for deck enrichment service: batch dictionary lookup, batch example generation, single-card example, cancellation, Sentry reporting
- `e2e/tests/*.spec.ts` — Playwright E2E tests: app loading, video flow, text flow, demo flow, word popup, flashcard review, add-to-deck, settings features, subscription/paywall, security headers (CSP compliance), edge cases

**Testing patterns:**
- **Server integration tests** mock at the `media.js` boundary via `vi.mock()` — real Express routing, sessions, SSE, and chunking run against fake yt-dlp/Whisper/GPT responses. Auth is also mocked (`req.uid = 'test-user'`).
- **E2E tests** intercept all network via Playwright's `page.route()` in `e2e/fixtures/mock-routes.ts` — `setupMockRoutes(page, { cached: true })` sets up mock API responses. Firebase is blocked (auth falls back to `useAuthE2E`, deck to localStorage).
- **E2E auth bypass**: build-time `VITE_E2E_TEST` flag swaps real `useAuth` for a stateful mock. Tests can set `window.__E2E_NO_AUTH = true` via `addInitScript()` to start logged-out.
- **Test isolation**: `clearAllCostsForTesting()` resets usage Maps; rate limiters disabled when `process.env.VITEST` is set.
- **Frontend unit tests** use `@testing-library/react` + jsdom; setup file imports `@testing-library/jest-dom/vitest` matchers.

## Related Documentation

- `ARCHITECTURE.md` — Sequence diagrams for auth, video analysis, chunk download, flashcard, text mode, rate limiting flows
- `CLAUDE_DOCS/` — Feature-specific docs (progressive disclosure format, one file per feature)
- `README.md` — User-facing docs with feature overview, API costs, and quick start

## Setup

1. `npm install && npm run server:install`
2. `brew install yt-dlp ffmpeg`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RaggedR) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
