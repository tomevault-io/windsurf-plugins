---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Run everything (recommended for local dev)

```bash
npm run dev          # starts backend (dotnet watch) + frontend (expo) concurrently
```

### Backend only

```bash
cd OpenRestoApi
dotnet watch run     # hot reload on :8080
dotnet test          # all backend tests
dotnet test --filter "FullyQualifiedName~BookingServiceTests"  # single test class
```

### Frontend only

```bash
cd openresto-frontend
npm run web          # Expo web dev server on :8081
npm test             # Jest unit tests
npm test -- --testPathPattern=BookingForm  # single test file
npm test -- --coverage  # coverage report
npm run test:e2e     # Playwright E2E tests
npm run check        # prettier + eslint (what CI runs)
npm run lint:fix     # auto-fix lint issues
```

### Docker (full stack through nginx)

```bash
docker compose up    # full stack on localhost:5062
```

## Architecture

Three-container stack: **Nginx** (`:5062`) → routes `/api/*` to **ASP.NET Core** (`:8080`) and `/*` to **Expo/React Native** (`:8081`). A shared Docker volume (`media_data`) serves uploaded images at `/media/`.

### Backend — Clean-ish layered architecture

```
OpenRestoApi/
├── Controllers/          # Thin HTTP layer — validate auth, call services, return DTOs
├── Core/
│   ├── Domain/           # Plain C# entities (Restaurant, Booking, Table, Section, …)
│   ├── Application/
│   │   ├── Services/     # Business logic (BookingService, AvailabilityService, AdminService, …)
│   │   ├── Interfaces/   # Contracts for repos, email, clock, holds
│   │   ├── DTOs/         # Request/response shapes
│   │   └── Mappings/     # Mapperly source-gen mappers (no AutoMapper)
└── Infrastructure/
    ├── Persistence/      # EF Core + SQLite (AppDbContext, repositories)
    ├── Holds/            # In-memory table hold service (singleton ConcurrentDictionary)
    ├── Email/            # MailKit SMTP wrapper
    ├── Cookies/          # Encrypted HttpOnly cookie for recent bookings (DataProtection)
    └── Auth/             # JWT generation helpers
```

**Key conventions:**

- All `DateTime` values are stored and passed as **UTC**. EF Core value converters enforce this globally in `AppDbContext`. Restaurant-local times are converted using the restaurant's IANA `Timezone` field only at display/availability-calculation time.
- `OpenDays` is a comma-separated string of ISO 8601 day numbers (`1`=Monday … `7`=Sunday).
- `HoldService` is a **singleton** in-memory store — appropriate for single-instance deployment. Holds expire after 5 minutes. If you need multi-instance, swap for Redis.
- The OpenAPI spec (`/openapi/v1.json`) is only exposed when `ASPNETCORE_ENVIRONMENT=Development`. The dev nginx template (`nginx/default.conf.template`) proxies `/openapi/` to the backend for ZAP CI scanning; the prod nginx (`nginx-vps/`) does not.
- **EF migrations with running dev server**: exe is locked, so use `dotnet ef migrations add <Name> --no-build`. If obj/ DLLs are stale the generated `Up()` will be empty — write it manually.
- **Cross-platform image generation**: use `Magick.NET-Q8-AnyCPU` (ships Linux x64 native libs, no apt-get needed). Never add `Svg` (SVG.NET) — it uses `System.Drawing.Common` which is Windows-only in .NET 7+.

### Frontend — Expo Router file-based routing

```
openresto-frontend/
├── app/
│   ├── (user)/           # Customer-facing: index (search), book, lookup
│   └── (admin)/          # Admin dashboard, bookings list, settings
├── api/                  # Typed fetch wrappers (one file per resource: restaurants, bookings, holds, …)
├── components/
│   ├── booking/          # BookingForm, PopularTimesPicker, HoldStatusBanner, useTableHold
│   ├── restaurant/       # RestaurantCard (home page tiles)
│   └── admin/            # Dashboard, tables, settings components
├── context/
│   ├── BrandContext      # Fetches /api/brand on mount; provides appName + primaryColor globally
│   └── ThemeContext
└── hooks/                # useColorScheme, etc.
```

**Key conventions:**

- `EXPO_PUBLIC_API_URL` drives all API calls. In Docker it is `/api` (relative, goes through nginx). In standalone dev it is `http://localhost:5062`. The `buildEndpoint` helper in `BrandContext` normalises both forms.
- Availability is fetched per `(restaurantId, date, seats)`. The API returns 30-minute slots with `{ time, isAvailable, availableTableIds, category }`. `PopularTimesPicker` shows only `isAvailable: true` slots; closed days return an empty slots array from the backend.
- Table holds flow: frontend calls `POST /api/holds` → backend validates open hours + pause state + conflict-checks → returns a `holdId` + expiry. The `holdId` must be included in the subsequent `POST /api/bookings` request.
- **Chrome favicon caching**: never update `<link rel="icon">` href in-place — Chrome ignores it. Remove all existing favicon links then append a fresh `<link>` element to force re-read.
- **PWA manifest URL**: must remain a same-origin HTTP(S) URL. Replacing `<link rel="manifest">` href with a `blob:` URL silently breaks Chrome's PWA installability check.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [karanshukla/openresto](https://github.com/karanshukla/openresto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
