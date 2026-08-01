---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`valhalla-web` (package name) is the ReactJS demo app that runs on https://valhalla.openstreetmap.de. It is a SPA frontend over the [Valhalla](https://github.com/valhalla/valhalla/CLAUDE.md) routing engine — it builds Valhalla `/route`, `/isochrone`, `/locate`, and `/height` requests and renders results on a MapLibre map. There is no backend in this repo.

## Commands

```bash
npm run dev            # Vite dev server on http://localhost:3000 (alias: npm start)
npm run build          # Vite production build → ./build (NOT ./dist)
npm run preview        # Serve the production build

npm test               # Vitest (watch mode); single test: npx vitest run path/to/file.spec.ts
npm run test:coverage  # Vitest with v8 coverage
npm run test:e2e       # Playwright (chromium + firefox); auto-starts dev server if not running
npm run test:e2e -- --project=chromium   # Single browser
npm run test:e2e:ui    # Playwright Test UI
npm run test:e2e:headed -- --project=firefox

npm run lint           # ESLint
npm run typecheck      # tsc --noEmit
npm run prettier       # Format
npm run check          # prettier:check && lint  (run before opening a PR)
npm run check:deps     # taze: list outdated deps interactively
```

Husky `pre-commit` runs `npm run typecheck && npx lint-staged` (eslint --fix on `*.{js,jsx,ts,tsx}`, prettier on `*.{json,md,scss,yaml,yml}`). CI (`.github/workflows/playwright.yml`) runs typecheck → lint → vitest → playwright (chromium only) on every PR.

## Tech stack

- **React 18** + **TypeScript** (strict, `noUncheckedIndexedAccess`, `verbatimModuleSyntax`) + **Vite 7**
- **TanStack Router** (code-based, not file-based — see `src/routes.tsx`)
- **TanStack Query** for all Valhalla/Nominatim fetches
- **Zustand** + `immer` + `devtools` middleware for client state (3 stores in `src/stores/`)
- **Tailwind CSS v4** via `@tailwindcss/vite` + **shadcn/ui** (style `new-york`, base `slate`, lucide icons; see `components.json`)
- **maplibre-gl** + **react-map-gl** + `@watergis/maplibre-gl-terradraw` for drawing exclude-polygons
- **react-day-picker** (used by the shadcn `Calendar` primitive that powers the date/time button)
- **date-fns** for formatting
- **zod** for env/search-param/URL validation
- Path alias: `@/*` → `src/*`

## Architecture

### Entry & routing

```
src/index.tsx                      Mounts <RouterProvider> wrapped in TanStackQuery <Provider>
└─ src/routes.tsx                  Defines the router (code-based)
   └─ rootRoute → RootComponent    Renders <Outlet/> + dev-only TanStack devtools
      ├─ '/'                       beforeLoad redirects to '/directions'
      └─ '/$activeTab'             component=<App/>; validateSearch=zodValidator(searchParamsSchema)
                                   activeTab ∈ {'directions','isochrones','tiles'}; invalid → redirect
```

`<App/>` (`src/app.tsx`) wraps everything in `MapProvider` and renders three siblings: `MapComponent`, `RoutePlanner`, `SettingsPanel`, plus a sonner `<Toaster/>`.

URL search params are the source of truth for `profile` (costing model) and `style` (map style); a `retainSearchParams` middleware keeps them across tab switches. Schema is in `src/utils/route-schemas.ts`.

The Vite `base` is derived from `package.json` `homepage` (see `vite.config.ts → getBaseUrl()`), and the router uses `import.meta.env.BASE_URL`. The PR-preview workflow rewrites `homepage` before building so the bundle is served from `/{PR_NUMBER}/`.

### State

Three Zustand stores, each with `immer` + `devtools`:

- `src/stores/common-store.ts` — settings panel/directions panel open state, costing settings, dateTime, map-ready flag. `Profile` enum and `profileEnum` zod schema live here.
- `src/stores/directions-store.ts` — waypoints (with geocode results), route results, highlighted maneuver, optimized-route flag, active-route index.
- `src/stores/isochrones-store.ts` — input/result, range/interval/denoise/generalize, color palette, opacity.

Server-state lives in TanStack Query. The global `QueryClient` (`src/lib/tanstack-query/root-provider.tsx`) sets `refetchOnWindowFocus: false`, `retry: 1`, `staleTime: 5min`, `gcTime: 10min`. Query hooks are in `src/hooks/use-*-queries.ts`. They read inputs directly from Zustand stores via `useStore.getState()` and from the router via `router.state.location.search` rather than parameters — keep that pattern when adding new queries.

### Components

- `src/components/map/` — MapLibre map. `index.tsx` is the orchestrator; `parts/` holds map sublayers (route lines, isochrone polygons, hover popups, draw controls, marker icons). `valhalla-layers.ts` defines internal Valhalla edge/node/shortcut/access-restriction MVT layer IDs.
- `src/components/directions/`, `src/components/isochrones/`, `src/components/tiles/` — the three tab panels.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [valhalla/web-app](https://github.com/valhalla/web-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
