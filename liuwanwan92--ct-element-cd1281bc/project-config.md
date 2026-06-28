---
trigger: always_on
description: You are an expert in TypeScript, Angular, and scalable web application development. You write maintainable, performant, and accessible code following Angular and TypeScript best practices.
---

You are an expert in TypeScript, Angular, and scalable web application development. You write maintainable, performant, and accessible code following Angular and TypeScript best practices.

## TypeScript Best Practices

- Use strict type checking
- Prefer type inference when the type is obvious
- Avoid the `any` type; use `unknown` when type is uncertain

## Angular Best Practices

- Always use standalone components over NgModules
- Must NOT set `standalone: true` inside Angular decorators. It's the default.
- Use signals for state management
- Implement lazy loading for feature routes
- Do NOT use the `@HostBinding` and `@HostListener` decorators. Put host bindings inside the `host` object of the `@Component` or `@Directive` decorator instead

## Components

- Keep components small and focused on a single responsibility
- Use `input()` and `output()` functions instead of decorators
- Use `computed()` for derived state
- Set `changeDetection: ChangeDetectionStrategy.OnPush` in `@Component` decorator
- Prefer inline templates for small components
- Prefer Reactive forms instead of Template-driven ones
- Do NOT use `ngClass`, use `class` bindings instead
- Do NOT use `ngStyle`, use `style` bindings instead

## Component Testing

- Test through the component's public API and rendered DOM, not private methods or internal implementation details
- Prefer creating the component directly with `TestBed`. Use signal `inputBinding` and `outputBinding` bindings instead of test host / wrapper components when wiring inputs and outputs
- Only introduce a test host / wrapper component when testing content projection, template composition, or integration behavior that cannot be expressed through direct bindings
- Keep test setup minimal. Mock only the component's direct dependencies and prefer lightweight spies or stubs over large testing modules
- Prefer assertions on user-observable behavior such as rendered text, attributes, ARIA state, CSS classes that are part of the public contract, and emitted outputs
- Prefer `await fixture.whenStable()` after interactions or async state changes instead of repeatedly calling `fixture.detectChanges()`. Use `fixture.detectChanges()` deliberately for the initial render or when the change detection boundary itself is under test
- Keep each test focused on a single behavior with clear arrange / act / assert phases
- Cover happy paths, boundary conditions, and regression-prone branches. Avoid broad snapshot-style assertions that do not explain the intended behavior

## Async Test Stabilization

- Prefer `await fixture.whenStable()` over repeated `fixture.detectChanges()` calls after interactions or async state changes
- `whenStable()` waits for pending microtasks, timers, and zone activity to settle, producing more reliable tests than manually pumping change detection
- Use `fixture.detectChanges()` deliberately for the initial render or when the change detection boundary itself is under test
- Do NOT chain multiple `detectChanges()` calls hoping to flush async work — use `whenStable()` instead

## Test Assertions

- Prefer Vitest's semantic assertion matchers over generic `.toBe()` with manual property access — they produce clearer failure messages and more readable tests
- Use `toHaveLength(n)` instead of accessing `.length` manually: `expect(items).toHaveLength(3)` not `expect(items.length).toBe(3)`
- Use `toContain(item)` instead of `expect(array.includes(item)).toBe(true)`
- Use `toMatchObject(subset)` to assert on a subset of properties instead of multiple individual `.toBe()` assertions

## Running Tests

Infer the correct test command from the file path being edited. After implementation changes, run the matching unit test command to verify.

### Unit Tests

All unit tests use **Vitest**. Do NOT use `npx ng test` or Karma/Jasmine commands.

| File path prefix                 | Command                      |
| -------------------------------- | ---------------------------- |
| `projects/element-ng/`           | `npm run lib:test`           |
| `projects/charts-ng/`            | `npm run charts:test`        |
| `projects/native-charts-ng/`     | `npm run native-charts:test` |
| `projects/maps-ng/`              | `npm run maps:test`          |
| `projects/dashboards-ng/`        | `npm run dashboards:test`    |
| `projects/element-translate-ng/` | `npm run translate:test`     |

To run a specific test file, use `--include` and `--no-watch`:

```shell
npm run lib:test -- --include='**/component-name/component-name.component.spec.ts' --no-watch
```

Only these CLI flags are supported: `--include` (glob filter) and `--no-watch` (run once).
Do NOT use `--reporter`, `--watch=false`, or any other flags.

### Schematics Tests

Run `npm run schematics:test` (uses Vitest). Config: `projects/element-ng/vitest.config.schematics.ts`.

### E2E Tests

E2E tests require Docker and a running dev server. They are executed via `./e2e-local.sh`.

```shell
# 1. Start the dev server (in one terminal)
npm run start -- --allowed-hosts true --host 0.0.0.0

# 2. For dashboards tests, also start dashboards artifacts
npm run dashboards-demo:build-and-run-all      # webpack

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liuwanwan92/ct-element-cd1281bc](https://github.com/liuwanwan92/ct-element-cd1281bc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
