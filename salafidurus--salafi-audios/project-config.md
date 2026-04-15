---
trigger: always_on
description: Purpose: give an AI coding agent just-enough context to be immediately productive and safe in this repository.
---

# Copilot instructions — Salafi Durus (monorepo)

Purpose: give an AI coding agent just-enough context to be immediately productive and safe in this repository.

## Quick orientation ✅

- Monorepo layout: `apps/` (api/web/mobile) + `packages/` (shared libraries) + `docs/` (source of truth).
- Canonical docs: the standard top-level docs in `docs/*.md` and the `AGENT.md` files at repo root and in each package/app (e.g. `apps/api/AGENT.md`). Start with `docs/README.md`.

## Non-negotiable guardrails 🛡️

- **Backend is authoritative.** Business rules, auth, and state transitions live in `apps/api`.
- **Authorization only on backend.** UI checks are UX-only, never security.
- **Offline = intent only.** Mobile records user intent in an outbox; backend authoritatively resolves state.
- **Media are references, not blobs.** Use presigned uploads; DB stores media keys/metadata (`packages/core-db/prisma`).
- **Monorepo boundaries:** apps → packages only. No app-to-app imports; avoid circular deps.

## Backend layering & examples 🔁

- Follow Interface → Application → Domain → Infrastructure (see `apps/api/AGENT.md` and `apps/api/src`).
- Shared contracts: `packages/core-contracts/src/types/*` and query helpers in `packages/core-contracts/src/query/`.
- DB & migrations: `packages/core-db/prisma/schema.prisma`, migrations in `packages/core-db/prisma/migrations/`.
- Client structure: `apps/web/AGENT.md` (app/core/features/shared) and `apps/mobile/AGENT.md` (outbox/sync patterns).

## Developer workflows — exact commands ▶️

- Install: `pnpm i`
- Run all in dev: `pnpm dev`
- Run single app: `pnpm dev:api`, `pnpm dev:web`, `pnpm dev:mobile`
- Build / Test / Lint / Typecheck: `pnpm build`, `pnpm test`, `pnpm lint`, `pnpm typecheck` (use Turbo filters to scope)
- API-only tests: `pnpm --filter api test`
- E2E (Playwright): `pnpm test:e2e`
- Shared contract updates: edit `packages/core-contracts` manually when backend response shapes change, then build/typecheck the package.

## Codegen & generated artifacts ⚠️

- Never hand-edit generated Prisma client output in `packages/core-db/src/generated/`; regenerate it from the Prisma schema.
- Shared API types are hand-written in `packages/core-contracts`; if types are wrong, fix them there and update backend usage to match.

## TDD policy — non-negotiable 🧪

This repo is TDD. **Write a failing test before writing implementation.** No exceptions.

### Workflow

```text
Red → Green → Commit  (test + implementation together, always)
```

Bug fixes start with a failing test that reproduces the bug.

### What to test

| Layer                              | Test type                          | Location                                           |
| ---------------------------------- | ---------------------------------- | -------------------------------------------------- |
| API service methods                | Unit — mock the repo               | `apps/api/src/modules/<mod>/<mod>.service.spec.ts` |
| Auth guard + permission boundaries | Unit + Integration                 | `apps/api/src/modules/auth/`                       |
| Domain store actions (Zustand)     | Unit — reset store state each test | `packages/domain-*/src/**/*.spec.ts`               |
| Pure utilities / helpers           | Unit                               | Co-located `.spec.ts` next to source file          |
| Route constant smoke tests         | Unit                               | `packages/core-contracts/src/routes.spec.ts`       |
| Critical user flows                | E2E (Playwright)                   | `apps/web/e2e/`                                    |

### What NOT to test

- Presentational-only React/RN components (no logic, no state).
- Trivial getters/setters or passthrough methods.
- Framework-provided wiring (NestJS DI, Expo Router file-based nav).
- Third-party library internals.

### Layer-specific rules

**API (NestJS):**

- Every service method that can throw a domain exception must have a unit test for that throw path.
- Every auth boundary (public vs. auth vs. admin permission) must have an integration test that verifies the correct HTTP status without a session vs. with one.
- Use `@nestjs/testing` + mocked repositories. Never connect to a real database in unit tests.

**Domain packages (`domain-progress`, `domain-playback`):**

- Zustand store actions are pure state machines — test every action with before/after state assertions.
- Reset the store between tests: `useStore.setState(initialState)`.

**Feature packages:**

- Only test exported utility functions and hooks that contain real logic.
- Do not test components purely for rendering; test behavior (e.g., a hook that computes a derived value).

**Web E2E (Playwright):**

- Cover: public pages load without errors, auth redirect fires for protected routes, sidebar navigates correctly.
- Do not cover: visual layout, font rendering, or UI polish.

### Test commands

```bash
pnpm test                                                     # all
pnpm --filter api test                                        # API only
pnpm --filter api test -- src/modules/scholars/scholars.service.spec.ts
pnpm --filter api test:watch -- src/modules/scholars/scholars.service.spec.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/salafidurus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
