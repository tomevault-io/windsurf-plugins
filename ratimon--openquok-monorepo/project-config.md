---
trigger: always_on
description: Test suite patterns — RBAC/user setup, Faker, anchor-before-failure, HTTP integration mocks (supertest), Flowcraft workflow unit tests, data cleanup
---


# Backend test suites

## 0. Logger noise under Jest

`utils/Logger.ts` suppresses **info** and **debug** (including `trace`) when `NODE_ENV` is `test` (set in `jest.env-setup.cjs`) or `JEST_WORKER_ID` is set, so Jest runs stay readable. **error** and **warn** still print. The check runs on each log call so hoisted imports cannot miss it.

To print verbose logs while debugging a suite: `BACKEND_TEST_VERBOSE_LOGS=true pnpm exec jest path/to.test.ts`.

### Expected `logger.warn` / `logger.error` in unit tests

Suites that **intentionally** trigger failure paths (mocked `reject`, invalid inputs, etc.) will execute real `logger.warn` / `logger.error` in production code. That is correct behavior; passing tests mean the service handled the failure as designed.

To avoid Jest printing those lines as if something were wrong, **stub the logger in that test file** (e.g. `beforeEach`: `jest.spyOn(logger, "warn").mockImplementation(() => {});` and the same for `error` where needed; `afterEach`: `jest.restoreAllMocks()`). Keep asserting on **return values, metrics, and repository calls**—not on console output.

## 1. Mocking roles (RBAC) in integration tests

Use the **Listing-style** pattern so platform admin, editor, and other app roles are set before tests run:

- **Create users** with `UserTestHelper.createVerifiedUserWithAuthAndDatabase(userData, options)`:
  - `userData`: `{ id: uuidv4(), email: '…@test.com', password, fullName }`.
  - `options`: `{ isPlatformAdmin?: boolean; isEmailVerified?: boolean }`.
  - **Platform admin**: pass `{ isPlatformAdmin: true, isEmailVerified: true }`. Only one user per suite should have `isPlatformAdmin: true`.
  - **Editor / normal user**: pass `{ isEmailVerified: true }` only (helper sets `is_super_admin: false` explicitly).
- **App roles** (editor, admin): assign via the API after user creation, e.g. `POST /users/:publicId/roles/editor` with the platform admin token. Do not only set DB flags; use the same API path the app uses.
- **Sign in** each user via `POST /auth/sign-in` and store the access token for `Authorization: Bearer <token>` in test requests.
- Run this setup in `beforeEach` so each test gets fresh users (and track all via `trackUser` so cleanup can remove them), or in `beforeAll` if the suite does not mutate shared users.

Reference: `backend/tests/integration/BlogRbac.integration.test.ts` (platform admin + editor + normal user, role assignment, then `afterAll` cleanup).

## 2. How we clean data

Cleanup lives in **UserTestHelper** (`backend/tests/helpers/userTestHelper.ts`). Respect FK order: delete child rows before parents (or rely on CASCADE where the schema defines it).

- **cleanAllStoredUsers()** — Cleans users tracked in `createdUserIds` (auth ids). Resolves to `public.users.id` list, then:
  1. Delete **blog_posts** for those user ids.
  2. Delete **organizations** (via user_organizations).
  3. Delete **organization_invites** by resolved emails.
  4. Delete auth users and **public.users** for each tracked auth id.
  5. Clear `createdUserIds`.
- **cleanTestUsersByEmailPattern()** — Cleans by email pattern (`@test.com`, `@example.com`):
  1. Delete **organization_invites** by pattern.
  2. Get user ids from **public.users** by pattern.
  3. Delete **blog_posts** for those ids, then **organizations** for those ids.
  4. Delete auth users and **public.users** rows; then sweep **auth.users** for any remaining matching emails.
- **cleanAll()** — Runs `cleanAllStoredUsers()` then `cleanTestUsersByEmailPattern()`. Use in suite **afterAll** when you want one full cleanup at the end (e.g. auth e2e).

**When to use which:** Use `cleanAllStoredUsers()` in **afterAll** (not afterEach) for RBAC/integration suites that create users in beforeEach and track them — Jest runs afterAll even when tests fail, so data is cleared once. Use `cleanAll()` in **afterAll** for e2e suites that also want pattern-based cleanup. Use **afterEach** + `cleanAllStoredUsers()` only when each test must start with a completely clean DB.

## 3. Clearing data for new tables

When you add a **new table** (e.g. in `backend/supabase/db/**/103_*_tables.sql` or similar), tests that reset or clean data must account for that table so test runs start from a clean state and do not leave rows that affect later tests.

- **Update the relevant test helper** (e.g. `backend/tests/helpers/userTestHelper.ts`) so that its cleanup paths also clear the new table where appropriate:
  - If cleanup is **by user/org** (e.g. "delete everything for these users"): add logic to delete rows in the new table that belong to those users or to orgs being removed (respect FKs and order: delete child rows before or with parent, or rely on CASCADE).
  - If cleanup is **by pattern** (e.g. "delete all test users by email pattern"): add a step to delete from the new table using the same pattern or the same list of identifiers (e.g. emails, org ids) before or after the existing cleanup.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ratimon/openquok-monorepo](https://github.com/Ratimon/openquok-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
