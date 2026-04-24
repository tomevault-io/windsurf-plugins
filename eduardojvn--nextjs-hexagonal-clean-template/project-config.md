---
trigger: always_on
description: A production-ready Next.js 15 starter that enforces Hexagonal (Ports & Adapters) / Clean Architecture with strict layer boundaries, full TDD support, and Shadcn UI. Use it as the foundation for any new project that requires maintainability at scale.
---

# CLAUDE.md — nextjs-hexagonal-clean-template

## What This Template Is

A production-ready Next.js 15 starter that enforces Hexagonal (Ports & Adapters) / Clean Architecture with strict layer boundaries, full TDD support, and Shadcn UI. Use it as the foundation for any new project that requires maintainability at scale.

---

## Commands

| Command              | What It Does                                          |
| -------------------- | ----------------------------------------------------- |
| `pnpm dev`           | Start the Next.js development server on :3000         |
| `pnpm test`          | Run all unit + server tests via Vitest                |
| `pnpm test:unit`     | Run client-side unit tests only (jsdom environment)   |
| `pnpm test:server`   | Run server-side tests only (Node environment)         |
| `pnpm test:coverage` | Run all tests with V8 coverage report                 |
| `pnpm test:e2e`      | Run Playwright end-to-end tests (requires dev server) |
| `pnpm lint`          | Run ESLint with Next.js rules + boundary enforcement  |
| `pnpm type-check`    | Run `tsc --noEmit` — no emit, just type validation    |

---

## Git Hooks (Husky)

| Hook         | Trigger      | What runs                                            |
| ------------ | ------------ | ---------------------------------------------------- |
| `commit-msg` | every commit | commitlint — validates message format                |
| `pre-commit` | every commit | lint-staged — ESLint + Prettier on staged files only |
| `pre-push`   | every push   | `pnpm test` — full unit + server test suite          |

E2E tests (Playwright) are intentionally excluded from pre-push — they require a running dev server.

### Conventional Commits format

```
<type>(<scope>): <subject>

feat(auth): add JWT token refresh
fix(todo): prevent double-complete on concurrent requests
test(domain): add edge cases for TodoTitle value object
chore(deps): upgrade next to 15.3.0
```

**Allowed types:** `feat` · `fix` · `docs` · `style` · `refactor` · `test` · `chore` · `perf` · `ci` · `revert`

Rules enforced by commitlint (`commitlint.config.ts`):

- type must be from the list above
- subject must be lowercase
- subject must not end with a period
- header max 100 chars

### Skip hooks (emergency only)

```bash
git commit --no-verify   # skip commit-msg + pre-commit
git push --no-verify     # skip pre-push tests
```

---

## Architecture Overview

### Layer Diagram

```
  ┌──────────────────────────────────────────────────────┐
  │                    Presentation                      │
  │   components/   +   app/actions/   (Server Actions)  │
  └────────────────────────┬─────────────────────────────┘
                           │ calls
  ┌────────────────────────▼─────────────────────────────┐
  │                   Infrastructure                     │
  │          src/infrastructure/  (adapters, repos)      │
  └────────────────────────┬─────────────────────────────┘
                           │ implements
  ┌────────────────────────▼─────────────────────────────┐
  │                    Application                       │
  │        src/application/  (use cases, ports/DTOs)     │
  └────────────────────────┬─────────────────────────────┘
                           │ uses
  ┌────────────────────────▼─────────────────────────────┐
  │                      Domain                         │
  │         src/domain/  (entities, value objects)       │
  └──────────────────────────────────────────────────────┘
```

**Dependency rule**: arrows point INWARD only. Inner layers never import from outer layers.

### What Lives in Each Layer

| Layer          | Path                          | Contents                                                              |
| -------------- | ----------------------------- | --------------------------------------------------------------------- |
| Domain         | `src/domain/`                 | Entities, Value Objects, Domain Errors, Repository interfaces (ports) |
| Application    | `src/application/`            | Use Cases, DTOs, Application Errors, Port interfaces                  |
| Infrastructure | `src/infrastructure/`         | Repository implementations (in-memory, DB, etc.), Composition Root    |
| Presentation   | `components/`, `app/actions/` | React Server/Client Components, Server Actions                        |

### Where Server Actions Live

Server Actions sit in `app/actions/` and belong to the **Presentation layer**. They:

1. Validate raw input with Zod
2. Call the relevant Use Case (Application layer)
3. Call `revalidatePath` / `revalidateTag` to bust the Next.js cache
4. Return a typed `ActionResult<T>` — never throw to the client

This keeps the Application and Domain layers free of Next.js dependencies.

### Composition Root

`src/infrastructure/composition/todo.composition.ts` wires repositories to use cases. This is the ONLY place where concrete adapter instances are created and injected. Swap implementations here — nowhere else.

---

## Vitest Environments


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EduardoJVN) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
