---
trigger: always_on
description: You are an expert in TypeScript, Angular, and scalable web application development. You write functional, maintainable, performant, and accessible code following Angular and TypeScript best practices.
---

You are an expert in TypeScript, Angular, and scalable web application development. You write functional, maintainable, performant, and accessible code following Angular and TypeScript best practices.

## Project Overview

Single-page Angular v22 demo app that **intentionally demonstrates why making Angular lifecycle methods `async` is an anti-pattern**. Angular Material + CDK v22, TypeScript 6 (strict), npm (Node version pinned in `.nvmrc`). A single route is defined in `src/app/app.routes.ts`; providers in `src/app/app.config.ts`; root component is `src/app/app.component.ts`.

The core thesis (explained in detail in `README.md`): Angular does not await async lifecycle hooks. Declaring `ngOnInit()` as `async` creates a misleading pattern where `await` appears to pause the component lifecycle, but it only pauses the async function itself — Angular moves on immediately.

The app demonstrates this with a deliberate broken sequence in `AppComponent.ngOnInit()`:

1. `ngOnInit()` is declared `async` and `await`s `FakeApiService.fakeApiCall1()` (`/api/fake1`).
2. Once that resolves, it fires calls 2–5 in parallel via `Promise.all(...)` (without awaiting it at the function level).
3. The template shows a `.loading` paragraph until all 5 calls complete, then switches to `.loading-done`. A `.ng-on-init-done` paragraph appears as soon as `ngOnInit` returns (after call 1, while calls 2–5 are still in-flight), proving Angular did not wait.

## Demo Preservation Rules ⚠️

**Do not refactor or "fix" `AppComponent`'s async patterns.** The broken code is the entire point of the demo.

- **Do NOT** make `ngOnInit()` non-async, remove `await` calls, or restructure the loading logic into a correct pattern. This would destroy what the app is demonstrating.
- **Do NOT** remove the `eslint-disable` comments in `app.component.ts`. They are intentional, necessary exceptions that allow the anti-pattern to compile for demonstration purposes:
  - `@typescript-eslint/no-misused-promises` — suppressed so `async ngOnInit()` is allowed.
  - `@angular-eslint/no-async-lifecycle-method` — suppressed so the async lifecycle method is allowed.
  - `@typescript-eslint/no-floating-promises` — suppressed for the unawaited `Promise.all(...)`.
- The "Never disable ESLint rules" linting guideline applies to all other files; `app.component.ts` is the sole intentional exception.
- Refer to `README.md` for the full explanation of the anti-pattern and recommended alternatives.

## Repository Map

- `src/app/app.component.{ts,html,scss,spec.ts}` — root component; owns all async logic and loading state (signals).
- `src/app/app.config.ts` — `ApplicationConfig` with `provideHttpClient()`, `provideRouter()`.
- `src/app/app.routes.ts` — single route `''` → `AppComponent`.
- `src/app/model/fake-model.ts` — `FakeModel` interface (`{ fake?: string }`). Re-exported from `src/app/model/index.ts`.
- `src/app/services/fake-api.service.ts` — `FakeApiService` with 5 `HttpClient.get()` methods for `/api/fake1`–`/api/fake5`. Re-exported from `src/app/services/index.ts`.
- `tests/` — Playwright end-to-end specs (`app.spec.ts`) and config (`playwright.config.ts`).
- `agent-instructions/source.md` — edit this, then sync (see below). Do not edit generated files directly.
- `scripts/sync-agent-instructions.mjs` — generator for the AI instruction files.

## Conventions

- File names use no `.component`/`.service`/`.directive` suffix for new files (Angular v20+ style).
- New components default to SCSS and external templates/styles.
- The generated instruction files (`AGENTS.md`, `.claude/CLAUDE.md`, etc.) are copies of this source — to change guidance, edit `agent-instructions/source.md` and re-sync (see below), never edit the copies.

## Common Commands

- Dev server: `npm run start` (http://localhost:4200/). Build: `npm run build` (prod) / `npm run build:dev`.
- Scoped unit test: `npm run test:unit -- --include src/app/app.spec.ts`. Full unit run: `npm run test:unit`.
- E2E: `npm run test:e2e`. Interactive UI mode: `npx playwright test --ui --config=./tests/playwright.config.ts`.
- Lint: `npm run lint` (auto-fix: `npm run lint:fix`). Format: `npm run prettier` (check: `npm run prettier:test`).
- CI (`.github/workflows/actions.yml`) runs, in order: sync check, lint, `npm run test`, Codacy coverage upload, build. Match this locally before pushing.

## TypeScript Best Practices

- Use strict type checking
- Prefer type inference when the type is obvious
- Avoid the `any` type; use `unknown` when type is uncertain

## Angular Best Practices

- Always use standalone components over NgModules
- Must NOT set `standalone: true` inside Angular decorators. It's the default in Angular v20+.
- Do NOT set `changeDetection: ChangeDetectionStrategy.OnPush` explicitly. `OnPush` is the default in Angular v22+.
- Use signals for state management
- Implement lazy loading for feature routes
- Do NOT use the `@HostBinding` and `@HostListener` decorators. Put host bindings inside the `host` object of the `@Component` or `@Directive` decorator instead
- Use `NgOptimizedImage` for all static images.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrlonis/example-angular-async](https://github.com/mrlonis/example-angular-async) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
