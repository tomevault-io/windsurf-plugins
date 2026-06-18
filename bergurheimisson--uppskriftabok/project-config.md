---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Backend (cookbook)**
```bash
cd backend
./mvnw spring-boot:run          # start dev server on :8080
mvn test                        # run all integration tests
mvn test -Dtest=RecipeControllerTest   # run a single test class
mvn test -Dtest=RecipeControllerTest#deleteRecipeReturns204AndIsGone  # single test method
```

**Auth service**
```bash
cd auth-service
./mvnw spring-boot:run          # start dev server on :8082
mvn test                        # run all integration tests (Zonky embedded DB, no Docker needed)
```

**Frontend**
```bash
cd frontend
npm run dev                     # Vite dev server on :5173, proxies /api/* to :8080
npm test                        # Vitest in watch mode
npm run test:run                # Vitest single run (CI)
npm run build                   # production build → dist/
```

**Mock mode** (frontend without a running backend):
```bash
# create frontend/.env.development.local containing:
VITE_USE_MOCK=true
npm run dev
```
The Vite config rewrites `../api` imports to `api.mock.js` at build time — there are no runtime flag checks in components.

**Deploy**
```bash
# Frontend change only
cd frontend && npm run build && docker compose restart nginx

# Backend change
docker compose up -d --build backend

# Auth-service change
docker compose up -d --build auth-service

# Both/all
cd frontend && npm run build && cd .. && docker compose up -d --build
```

## Architecture

Full details are in `ARCHITECTURE.md`. Key points for day-to-day work:

**Request path:** Browser → Cloudflare Tunnel → cloudflared → Nginx :80 → Spring Boot :8080 → PostgreSQL :5432. Write requests (`POST`/`PUT`/`PATCH`/`DELETE`) pass through an `auth_request` subrequest to auth-service :8082 before reaching the backend. In dev, Vite proxies `/api/*` directly to `:8080`, skipping Nginx and auth.

**Auth-service layout** (`is.bergur.auth`):
- `controller/` — `AuthController` (login, refresh, logout, validate) and `AdminController` (user CRUD)
- `service/` — `JwtService`, `UserService` (BCrypt), `RefreshTokenService` (rotation)
- `security/` — `SecurityConfig` (Spring Security 6, stateless) and `JwtAuthFilter` (`OncePerRequestFilter`)
- `model/` — `User`, `RefreshToken` JPA entities
- Flyway history table: `auth_flyway_schema_history` (separate from cookbook's `flyway_schema_history`)

**Backend layout** (`is.bergur.uppskriftabok`):
- `controller/` — HTTP mapping only, delegates to service
- `service/` — business logic; `RecipeService`, `GroceryService`, `ImportService`, `IngredientParser`
- `repository/` — Spring Data JPA interfaces
- `model/` — JPA entities; `Recipe` stores `ingredients` as JSONB (`List<Ingredient>`), not a join table; `instructions` is plain `TEXT`

**Frontend layout** (`src/`):
- `pages/` — route-level components (`Home`, `Recipe`, `Add`, `Edit`, `Grocery`, `Login`, `Admin`)
- `components/` — reusable (`RecipeCard`, `AddRecipeForm`, `CookMode`, `ServingScaler`, `GroceryList`)
- `context/AuthContext.jsx` — JWT stored in React state (memory only, never localStorage); silent refresh on mount
- `api.js` — real API client; module-level `_token` variable injected as `Authorization: Bearer` header on all write requests; `api.mock.js` — in-memory mock with the same interface
- `utils/fractions.js` — client-side fraction formatting (serving scaler math stays in the browser)

**Routes:** `/` Home, `/recipe/:id` Recipe detail, `/add` Add, `/recipe/:id/edit` Edit, `/grocery` Grocery list, `/login` Login, `/admin` User management (ADMIN only).

`AddRecipeForm` serves both create and edit: pass `initialRecipe` prop to pre-fill fields and switch to `PUT`. Without the prop it renders as "Add Recipe" and calls `POST`.

## Testing

**Backend:** integration tests only — no unit tests for individual classes. All tests extend `AbstractIntegrationTest` which provides `@SpringBootTest(webEnvironment=RANDOM_PORT)` + `@AutoConfigureEmbeddedDatabase` (Zonky embedded PostgreSQL — no Docker required). Use `TestRestTemplate` for HTTP-level assertions; `@BeforeEach` cleans the repo.

**Frontend:** Vitest + React Testing Library. Tests render components via their public output, not implementation details. Mock API is used throughout — do not introduce `jest.mock` for the API module.

**Auth-service tests** follow the same pattern as the cookbook backend: integration tests only, `@SpringBootTest(webEnvironment=RANDOM_PORT)` + `@AutoConfigureEmbeddedDatabase` (Zonky). No Docker needed. Use `TestRestTemplate` for HTTP-level assertions.

**TDD is mandatory.** Write a failing test first, then the minimum code to pass it.

## Key Constraints and Non-obvious Decisions

**JSONB for ingredients:** Recipes can be stored in any language (Icelandic, English, etc.). "Hveiti" and "Flour" are the same ingredient — normalising into a relational table would require a multilingual dictionary. JSONB keeps ingredients self-contained per recipe. Never introduce a `recipe_ingredients` table.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BergurHeimisson/uppskriftabok](https://github.com/BergurHeimisson/uppskriftabok) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
