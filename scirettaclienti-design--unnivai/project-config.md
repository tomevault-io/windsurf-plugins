---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start dev server on port 5173 (strict port)
npm run build     # Production build
npm run lint      # ESLint check
npm run preview   # Preview production build
```

```bash
npm test                  # watch mode (Vitest)
npm run test:run          # single pass (CI)
npm run test:ui           # browser UI at localhost
npm run test:coverage     # coverage report (v8)
```

**Stack:** Vitest 3 + jsdom + @testing-library/react + @testing-library/jest-dom
**Location:** `src/__tests__/` mirrors `src/` structure.
**Mocks:** `src/test/mocks/supabase.js` exports `createQueryBuilder()`, `createSessionMock()`, `NO_SESSION`. Use `vi.mocked(supabase.from).mockReturnValue(createQueryBuilder({data, error}))` per test.
**Setup:** `src/test/setup.js` stubs `navigator.geolocation` and silences `console.error/warn` from dataService migration warnings.
**fetch mocking:** Use `vi.stubGlobal('fetch', vi.fn().mockResolvedValue({ok, json}))` for Nominatim calls in userContextService tests. Always pair with `vi.unstubAllGlobals()` in `afterEach`. Use `vi.resetAllMocks()` (not `clearAllMocks`) in `beforeEach` so mock implementations don't bleed between tests.

## Environment Variables

Copy `.env.example` to `.env` and fill in:
```
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=
VITE_MAPBOX_TOKEN=
VITE_OPENAI_API_KEY=   # Optional: enables real AI itinerary generation
```

All env vars must be prefixed with `VITE_` to be accessible via `import.meta.env`.

## Architecture Overview

**DoveVai** is a mobile-first React 19 + Vite SPA for Italian tourism. It connects travellers (esploratori), local guides, and businesses. The UI language is Italian.

### Provider Stack (outermost → innermost)
```
QueryClientProvider (React Query, 5-min stale cache)
  └── AuthProvider (Supabase auth + role)
        └── CityProvider (active city, GPS vs manual)
              └── Router → ErrorBoundary → Suspense → Routes
```

### Role-Based Routing

`RootDispatcher` (at `/`) redirects authenticated users by role:
- `explorer` / `user` → `/dashboard-user`
- `guide` → `/dashboard-guide`
- `business` → `/dashboard-business`
- unauthenticated → `/` (Landing page)

`RoleGuard` wraps protected route groups and enforces role access. Role is read from `user.user_metadata.role`, with fallback to `localStorage.getItem('unnivai_role')`.

All pages are lazy-loaded via `React.lazy()`.

### Path Alias

`@` resolves to `./src/` (configured in `vite.config.js`).

### Key Contexts

| Context | File | What it provides |
|---|---|---|
| `AuthContext` | `src/context/AuthContext.jsx` | `user`, `role`, `isAuthenticated`, `signOut`, `resetPassword`, `isPasswordRecovery` |
| `CityContext` | `src/context/CityContext.jsx` | `city` (default `'Roma'`), `setCity` (marks as manual), `isManual`, `resetToGPS` |

### Key Services (singleton class instances)

| Service | File | Responsibility |
|---|---|---|
| `dataService` | `src/services/dataService.js` | Supabase CRUD: tours, bookings, favorites, notifications, activities, businesses. Exposes `mapTourToUI()` which normalizes DB rows to the UI contract. |
| `aiRecommendationService` | `src/services/aiRecommendationService.js` | AI itinerary generation. Uses OpenAI GPT (if `VITE_OPENAI_API_KEY` set) or falls back to local mock data. Also handles business analysis via vision model. |
| `userContextService` | `src/services/userContextService.js` | Resolves effective city from manual override → GPS → Supabase profile → localStorage. Does reverse geocoding via Nominatim (OSM). |
| `weatherService` | `src/services/weatherService.js` | Provides weather data for the current city. |

### Key Hooks

- `useUserContext()` — composes GPS, auth, and CityContext into a single user context object (`city`, `firstName`, `temperatureC`, `toursCount`, `isGuest`). Used by dashboards and TopBar.
- `useEnhancedGeolocation()` — wraps browser GPS API, reverse geocodes to city name via Nominatim, falls back to Roma on error/denial.
- `useUserNotifications()` — `src/hooks/useUserNotifications.js`

### Supabase Schema

All migrations live in `supabase/migrations/`. Apply them in filename order. RLS base policies in `supabase/rls_policies.sql`.

**Canonical tables and their authoritative columns:**

| Table | PK | Key columns | Notes |
|---|---|---|---|
| `tours` | `id` | `city`, `guide_id`, `is_live`, `price_eur`, `duration_minutes`, `image_urls` | `price_eur` added by `migration_add_filters.sql` |
| `profiles` | `id` (= auth user id) | `full_name`, `avatar_url`, `bio`, `current_city`, `username` | Synced with `auth.users` |
| `bookings` | `id` | `tour_id`, `user_id`, `status` (`pending_request`→`accepted`/`declined`) | |
| `guide_requests` | `id` | `user_id`, `guide_id`, `tour_id`, `status` (`open`/`accepted`/`declined`/`completed`), `created_at`, `city`, `request_text`, `user_name`, `duration`, `category`, `notes` | Columns added across two migrations |
| `guides_profile` | `id` | `user_id` (unique), `license_number`, `piva`, `bio`, `status`, `type` (`pro`/`host`), `operating_cities[]` | Created in `20260303` migration |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scirettaclienti-design/unnivai](https://github.com/scirettaclienti-design/unnivai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
