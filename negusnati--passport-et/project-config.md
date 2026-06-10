---
trigger: always_on
description: This document summarizes the strategy for turning the existing Laravel/Inertia project inside `src/` into a high-performance API platform that powers an external React client. It distills the current codebase, highlights bottlenecks, and presents best practices for API design, performance tuning, search, and phone-number-first authentication.
---

# AGENTS

This document summarizes the strategy for turning the existing Laravel/Inertia project inside `src/` into a high-performance API platform that powers an external React client. It distills the current codebase, highlights bottlenecks, and presents best practices for API design, performance tuning, search, and phone-number-first authentication.

## 1. Current State Snapshot
- Web-only routing still powers the legacy UI via `src/routes/web.php`, while a new versioned scaffold in `src/routes/api.php` exposes stub JSON endpoints for the upcoming API without affecting existing Inertia responses.
- Core data is stored in `p_d_f_to_s_q_lites`, surfaced through `App\Models\PDFToSQLite` with wide-open `$guarded = []`, and queried directly in controllers with `like` filters and `simplePaginate`.
- Caching already appears in controllers (`Cache::remember` for passport lists, city lookup) and in the closure front page count, but cache invalidation is missing.
- `RateLimitMiddleware` implements per-plan limits on top of `Cache`, but does not guard against unauthenticated visitors (null `$user`).
- Auth still relies on the default Laravel email/password stack with Inertia-driven UI.

### Current API Targets (Phase 0 Baseline)
- **Controllers & Flows**: `src/app/Http/Controllers/PassportSearchController.php` renders `Passport/Show` (search) and `Passport/TableView` (all passports) and queries `PDFToSQLite` with prefix `LIKE` filters, limiting to 60 results or `simplePaginate(50)` for archives. `FilterByCityController` caches distinct locations (`cities`) for an hour and memoizes per-location pagination results; `SubscriptionController` assigns `plan` values on the authenticated user before redirecting to the dashboard.
- **Caching Touchpoints**: Global count cache on `/` (`passport_count` key), per-location caches (`passports_location_{location}_page_{n}`), `cities` lookup cache, and the legacy `RateLimitMiddleware` that increments counters without null-guarding `$user`. New API limiter (`api.v1.default`) returns JSON errors and uses user-id/IP keys.
- **Data Model & Indexes**: `p_d_f_to_s_q_lites` columns include `no`, `firstName`, `middleName`, `lastName`, unique `requestNumber`, `location`, and `dateOfPublish` with timestamps. Indexed fields: `location`, `firstName`, composite `firstName+middleName+lastName`, and `requestNumber`.
- **Authentication Stack**: `config/auth.php` keeps the default `web` session guard backed by `App\Models\User`; Laravel Breeze routes in `routes/auth.php` handle registration, login, password reset, and email verification. No API guard or token issuing is configured yet.
- **Outstanding Baseline Checks**: Dockerized commands are partially complete. `composer validate` and `php artisan test --testsuite=Unit` were executed successfully inside the Docker stack on 2025-09-23. `npm run lint` still needs to be run (command: `docker compose run --rm npm "npm run lint"`).

### Phase 2 Progress Log
- Shared domain model `App\Domain\Passport\Models\Passport` now owns search scopes (`filter`, `sort`, `limitForSearch`) with mass-assignment protection and `dateOfPublish` casting. Legacy `App\Models\PDFToSQLite` extends it to keep existing references working.
- `PassportSearchController` delegates search logic to `App\Actions\Passport\SearchPassportsAction`, reducing controller complexity and preparing for API reuse. Inputs are normalized via `PassportSearchParams` DTO, enabling consistent casing, date handling, and pagination defaults.
- Centralized filter metadata and cache helpers live in `App\Support\PassportFilters` and `App\Support\CacheKeys`; search caching uses hashed keys with 60-second TTL (overrideable later).
- New Pest unit tests under `tests/Unit/Domain/Passport/PassportSearchTest.php` cover request-number filtering, date/location filters, and the action’s pagination behaviour to guard against regressions as the API layer adopts the service.

### Phase 3 Progress Log
- `/api/v1/passports` now resolves through `PassportController`, reusing `SearchPassportsAction` with API-specific validation (`SearchPassportRequest`) and returning `PassportCollection` resources that expose consistent `data`, `meta`, `links`, and a `filters` echo for the React client.
- `/api/v1/passports/{id}` surfaces individual records through `PassportResource`, while `/api/v1/locations` serves cached distinct locations (`api.v1.locations` key, 5-minute TTL).
- `PassportResource` now shapes fields for API consumers (normalized names, ISO timestamps) and `PassportCollection` adds `meta.has_more` for paginated responses while tracking counts for non-paginated searches.
- Feature coverage added in `tests/Feature/Api/PassportApiTest.php` validating pagination metadata, filtering semantics, detail responses, and location listings. Run via `docker compose exec php php artisan test --testsuite=Feature --filter=Api`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NegusNati/passport.et](https://github.com/NegusNati/passport.et) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
