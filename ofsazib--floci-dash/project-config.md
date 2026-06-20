---
trigger: always_on
description: **Every build, run, stop, and Docker operation MUST use `make` commands.** Never call `docker compose` or `pnpm run` directly for project operations.
---

# Agent Instructions — Floci Dashboard

## MANDATORY: Use the Makefile

**Every build, run, stop, and Docker operation MUST use `make` commands.** Never call `docker compose` or `pnpm run` directly for project operations.

| Do | Don't |
|----|-------|
| `make up-bg` | ~~`docker compose up --build -d`~~ |
| `make down` | ~~`docker compose down`~~ |
| `make rebuild` | ~~`docker compose build ... && docker compose up -d`~~ |
| `make logs` | ~~`docker compose logs -f`~~ |
| `make typecheck` (native) | OK for local dev, but `make typecheck-docker` in Docker |
| `make help` | — |

Run `make help` to see all available commands.

## MANDATORY: Update README After Changes

**After implementing any feature, fixing a bug, or making structural changes, the agent MUST update `README.md`** to keep it accurate for open-source users. Specifically:

1. **Supported Services table** — Add new services when fully implemented
2. **Project Structure** — Add new directories/files when created
3. **Features** — Add new user-facing capabilities
4. **Commands** — If new make targets or scripts are added
5. **Environment Variables** — If new env vars are introduced

The README is the first thing users see. Keep it comprehensive, current, and well-formatted.

## MANDATORY: Plan & Tracker

**Every agent MUST follow the implementation plan in `PLAN.md`.**

1. **Read PLAN.md** before starting any work — it contains the full implementation plan with phase-by-phase task breakdown
2. **Check the PROGRESS TRACKER** in PLAN.md to see what's done, in progress, and pending
3. **Update the tracker** when you start a task (Pending -> In Progress) and when you complete it (In Progress -> Done + date)
4. **Never mark a task Done** without running `make typecheck` successfully first
5. **Never skip verification** — each service phase ends with a typecheck + build verification step

The tracker uses these status values: `Done`, `In Progress`, `Pending`, `Blocked`

## MANDATORY: Tests & Codecov Coverage

**Every feature implementation MUST include tests before committing.** No feature is "done" without tests.

### Required steps after implementing any feature:

1. **Write backend route tests** (`src/backend/routes/aws/{service}.test.ts`)
   - Mock the AWS SDK client and all command constructors using the `vi.hoisted` + `createCmd` pattern (see `kms.test.ts` or `ecs.test.ts` for reference)
   - Test every endpoint: happy path, empty results, error/400 validation cases
   - Target: **>90% statement coverage** on new route files

2. **Write frontend hook tests** (`src/frontend/hooks/use{Service}.test.ts`)
   - Mock `api()` from `../lib/client`
   - Test every query hook: correct URL called, `enabled` gate when param is null
   - Test every mutation hook: correct method/URL/body, invalidation on success
   - Target: **100% statement coverage** on new hook files

3. **Write component/page tests** for non-trivial UI components
   - Use happy-dom environment (`// @vitest-environment happy-dom`)
   - Use `createWrapper()` from test helpers for React Query context
   - Test user flows: render, click, fill forms, verify API calls

4. **Run coverage verification before committing:**
   ```bash
   npx vitest run --coverage
   ```
   - Verify new files have **>90% statement coverage**
   - Verify overall coverage **does not decrease** below current thresholds in `vitest.config.ts`
   - If coverage drops, add more tests — do not lower thresholds

5. **Codecov best practices:**
   - `codecov.yml` enforces a **75% patch target** — new code must meet this bar
   - Test both success AND error branches (e.g., empty arrays, missing params → 400)
   - Cover edge cases: URL encoding, optional params, default values
   - Never skip tests to save time — incomplete test coverage is technical debt
   - Prefer many small focused tests over one large test
   - Each test should verify one behavior (`it("does X when Y")`)

### Existing test patterns to follow:

| Pattern | Reference file |
|---------|---------------|
| Backend route mock | `src/backend/routes/aws/kms.test.ts` |
| ECS backend mock (`create()` factory) | `src/backend/routes/aws/ecs.test.ts` |
| Frontend hook test | `src/frontend/hooks/useKMS.test.ts` |
| ServicePage component test | `src/frontend/pages/ServicePage.test.tsx` |

## Project

Floci Dashboard is a Dockerized, full-stack web app providing an AWS Console-style UI for the Floci local AWS emulator. This project is open source — write code and docs accordingly.

- **Frontend:** React 19 + Cloudscape Design System + TanStack Query + React Router (HashRouter)
- **Backend:** Node.js 22 + Hono + @aws-sdk/client-* (all AWS SDK calls go through the backend, never the browser)
- **Infra:** Single Docker container, docker-compose pairs with Floci on port 4566

## Architecture Rules

1. **Zero Floci changes.** Dashboard uses only Floci's existing APIs. Never edit `../floci`.
2. **AWS SDK lives in the backend only.** The browser never imports @aws-sdk/client-*.
3. **Frontend calls /api/* routes on the dashboard backend.** Backend proxies to Floci or uses AWS SDK.
4. **Service-based vertical slices.** Each AWS service (S3, DynamoDB, etc.) gets its own backend route file.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ofsazib/floci-dash](https://github.com/ofsazib/floci-dash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
