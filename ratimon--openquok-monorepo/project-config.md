---
trigger: always_on
description: Backend API layer flow — routes → controller/data/errors/utils → service → repository; add e2e tests per feature
---


# Backend API layers (routes → controller → service → repository)

New API features follow this flow. Implement each layer in order and wire in index files.

## 1. Routes (`backend/routes/`)

- One router per domain (e.g. `UserRoute.ts`, `AuthRoute.ts`).
- For **public (unauthenticated) routes**, add the path to `publicPaths` in `backend/middlewares/core.ts` (see **backend-public-api-paths**).
- Compose: **auth middleware** (if protected), **validate\* middleware** (from `data/schemas`), **controller method**.
- Mount router in `routes/index.ts` under the API prefix (e.g. `apiRouter.use("/users", userRouter)`).

```ts
// UserRoute.ts
const auth = requireFullAuth(supabaseServiceClientConnection);
userRouter.get("/me", auth, userController.getProfile);
userRouter.put("/me/password", auth, validateUpdatePasswordMeRequest, userController.updatePasswordMe);
```

## 2. Data layer (`backend/data/`)

- **schemas/** — Zod schemas + `validateRequest(...)` middleware + **handler type** for controller typing.
  - Export the single middleware used by the route (e.g. `validateUpdatePasswordUserRequest`) and `export type validateXxxRequestHandler = typeof validateXxxRequest`.
  - Controller methods that have validation use that type: `updatePassword: validateUpdatePasswordUserRequestHandler = async (req, res, next) => { ... }`.
- **types/** — Shared API/domain types (e.g. `UserProfileResponse`). Optional.

## 3. Errors (`backend/errors/`)

- Domain errors that need a specific HTTP status: extend **AppError** (e.g. `UserError` → `UserNotFoundError`, `UserAuthorizationError`). ErrorController handles all `AppError` by `statusCode`; no per-error branches.
- Auth/validation/infra: use existing **AuthError**, **ValidationError**, **InfraError** as appropriate.

## 4. Utils (`backend/utils/`)

- **dtos/** — **API response shapes (camelCase DTOs), DB-aligned types with a `Like` suffix, and mappers** (e.g. `SocialPostDTO`, `SocialPostLike`, `PostDTOMapper.toDTO(...)`; `FeedbackLike`, `toFeedbackDTO`). Types such as **`SocialPostLike` / `FeedbackLike`** describe raw / persisted shapes (snake_case columns, join payloads) and live **here**, not in repositories. Repositories **import** those types from the appropriate DTO module for return types and casts; **do not** introduce a parallel `Row` duplicate name for the same shape.
- Some modules bundle related concerns in one file (e.g. `IntegrationDTO.ts`: `IntegrationLike` for DB rows plus `IntegrationCatalogDTO` / `IntegrationListDTO` for API responses).
- **Mapping is invoked in the controller**, not in the service (same as before).
- **valueObjects/** — Domain value objects (e.g. `UserId`) when you need validation/reuse. Optional.

## 5. Controller (`backend/controllers/`)

- Receives **Request, Response, NextFunction**. Cast to **AuthenticatedRequest** when using `req.user`.
- Call **service** (and optionally other services, e.g. AuthenticationService). Do not call repositories directly.
- **Map to DTOs in the controller** just before sending the response (e.g. pass service result into a DTO mapper, then `res.json({ data: dtos })`). Do not expect services to return API DTOs; services return **persistence-aligned `Like` types or domain types** from the persistence layer (via repositories), not camelCase API DTOs.
- **Create/update responses** — Use a consistent envelope: `{ success: true, data: { id: result.id }, message: "X created/updated successfully" }`. Return **201** for create and **200** for update. The service still returns full domain data; the controller exposes only `id` in `data` for create/update. Reference: `BlogController` (createBlogPost, updateBlogPost, createBlogTopic, updateBlogTopic), `ListingController` (createListing, updateListing).
- On validation/authorization/not-found: **`return next(new XxxError(...))`**. On unexpected errors: **`next(error)`** in catch. Do not throw in async handlers.
- Instantiate in `controllers/index.ts` with injected services; export the controller instance.

## 6. Service (`backend/services/`)

- Holds business logic; depends on **repositories** (and config). No Express types.
- Methods return persistence-aligned types (e.g. `SocialPostLike`, `OrganizationLike`), repository results, or domain types—not API DTOs. Import the matching symbols from **`utils/dtos`** when typing parameters or return values that mirror DB rows. The controller maps to API DTOs just before `res.json(...)`.
- Instantiate in `services/index.ts` with injected repositories; export instances.

### Cache (when the domain benefits from caching)

See **backend-service-cache** for full conventions (key naming like `LIST_BYUSERID` / `BY_ID`, explicit invalidation of read keys, CacheInvalidationService usage).

- **Dependency**: Inject **CacheService** and optional **CacheInvalidationService** from `connections`. Both optional so tests can omit them.
- **Key design**: Domain-scoped **`CACHE_KEYS`** and **TTL constant**. Name keys so scope is clear (e.g. `ORG_LIST_BYUSERID`, `BLOG_BYID`).
- **Read path**: **`cache.getOrSet(cacheKey, factory, ttl)`**; when `cache` is undefined, call repository directly.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ratimon/openquok-monorepo](https://github.com/Ratimon/openquok-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
