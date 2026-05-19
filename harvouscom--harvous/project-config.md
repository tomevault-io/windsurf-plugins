---
trigger: always_on
description: npm run dev              # Hono API (3001) + SPA (4322) — full-stack dev (API must be running for /api)
---

# Harvous for AI Agents

## Quick Commands

```bash
npm run dev              # Hono API (3001) + SPA (4322) — full-stack dev (API must be running for /api)
npm run dev:spa          # SPA only on port 4322 (proxies /api to 3001). API must be running or /api returns 500.
npm run dev:all          # Same as dev: API + SPA
npm run build            # Production build: inject SW version + build:api + build:spa (no Astro)
npm run build:api        # Bundle Hono API to netlify/functions/api.cjs
npm run db:sync          # Drizzle Kit push (sync schema to Supabase)
npm run db:push          # Drizzle Kit push (apply server/db/schema.ts to Supabase)
npm run db:check         # Pre-commit schema check (server/db/schema.ts)
npm run test:e2e         # Playwright e2e (join/invite flows)
npm run test:e2e:setup   # Seed e2e data then run e2e
npm run lighthouse:a11y  # Build SPA, vite preview, Lighthouse accessibility (must score 100); use `-- --skip-build` to skip build
npm run bible:generate -- NASB     # Generate NASB.json (NASB 1995) via Claude (needs ANTHROPIC_API_KEY in .env); resumes from partial
npm run bible:generate:all         # Generate NASB 1995 / CSB / AMP / MSG in sequence via Claude
npx tsx server/scripts/seed-bible-verses.ts NASB   # Import server/data/bibles/NASB.json (NASB 1995) into Supabase BibleVerses
```

**Clean new user (manual only):** The automatic dev-reset middleware was removed so production user data is never erased. To get "new user" state locally, call `POST /api/test/reset-to-new-user` (test route) when the API is running.

## Architecture Overview

**Harvous** is a Bible study notes app. Three-level hierarchy: Spaces → Threads → Notes. Data: Supabase Postgres (Drizzle ORM), schema in `server/db/schema.ts`.

- **Production frontend**: React SPA in `spa/src/`, built with Vite. Uses TanStack Router, React Query, Clerk React. Deployed as static `index.html` + hashed JS/CSS. This is what users see in production and in the PWA.
- **API backend**: Hono server in `server/` bundled as a single Netlify Function (`netlify/functions/api.cjs`). All `/api/*` requests are routed there by `public/_redirects`.
- **Shared React components**: `src/components/react/` are imported by the SPA. UI changes that must ship to production should be made in `spa/src/` or these shared components.
- **Auth**: Clerk. In the SPA, `@clerk/clerk-react`; env var `VITE_CLERK_PUBLISHABLE_KEY`.
- **Rich Text**: Tiptap editor in `src/components/react/TiptapEditor.tsx`.
- **Mobile bottom sheet**: [Vaul](https://github.com/emilkowalski/vaul) via `src/components/ui/drawer.tsx` (`BottomSheet.tsx`, `MobileNavigation.tsx`). Harvous keeps existing overlay/sheet CSS. Toast UI uses [Sonner](https://github.com/emilkowalski/sonner). Motion direction credits **[Emil Kowalski](https://emilkowal.ski/)**.

**Production API contract:** The API is built as a single file (`netlify/functions/api.cjs`); Netlify uses `node_bundler = "none"`, so there is no `node_modules` at function runtime. All dependencies must be bundled (do not add `--packages=external` to `build:api`). The DB client uses `postgres.js` which bundles cleanly for Netlify Functions. Before merging API-affecting branches, see [docs/CLEAR_SPLIT_MERGE_DELTA.md](docs/CLEAR_SPLIT_MERGE_DELTA.md) (pre-merge checklist).

## Project Structure

```
spa/                         # PRODUCTION FRONTEND (Vite SPA)
  src/
    layouts/                 # AppLayout.tsx (authenticated), AuthLayout.tsx
    pages/                   # DashboardPage, NotePage, ThreadPage, SpacePage, etc.
    hooks/queries/           # React Query hooks (useNote, useThread, useSpace, ...)
    router.tsx               # TanStack Router route definitions
    main.tsx                 # Entry point, global CSS imports
    lib/api.ts               # API client wrapper
    shims/                   # Shims (e.g. app-navigate for safeNavigate)
src/
  components/react/          # Shared React components (used by SPA)
  utils/                     # Shared utilities
  styles/                    # Vanilla CSS (imported by SPA)
server/
  db/                        # Drizzle schema (schema.ts), client (Supabase), dates
  routes/                    # Hono API routes
  utils/                     # Server-only utils (dashboard-data, user-cache, ...)
public/                      # Static assets, sw.js, manifest.json
```

## Code Style

- **TypeScript**: Strict mode, `@/` path aliases for imports (in both `src/` and `spa/` via vite resolve alias).
- **React Components**: Use hooks; shared ones in `src/components/react/` (PascalCase.tsx); SPA-specific in `spa/src/`.
- **CSS**: Semantic classes (no Tailwind), CSS variables for colors, organized by component.
- **Formatting**: Prettier (2 spaces, 120 char line width, trailing commas off).

## Key Patterns

- **CRITICAL — Production = SPA + Hono API.** For UI changes that must appear in production, edit `spa/src/` or shared `src/components/react/`.
- **Netlify build**: `npm run build` = inject SW + build:api + build:spa. Publish directory is `dist-spa/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harvouscom/harvous](https://github.com/harvouscom/harvous) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
