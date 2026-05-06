---
trigger: always_on
description: Playwright E2E test conventions — POM design, assertions, selectors, test structure
---

# Playwright E2E Conventions

## Imports

Tests import `test` and `expect` from the custom fixtures, not from `@playwright/test`:

```typescript
import { test, expect } from "../../fixtures";
```

This gives access to custom fixtures (workspace, app, api) and custom matchers (toShowToast).

## Page Object Model Rules

- POMs **never assert**. They return Locators or data. Assertions belong in test files.
- POMs **never sleep**. No `waitForTimeout`, no post-action delays.
- POMs **are thin**. A method is 1-5 lines. If longer, split it.
- POMs **don't compose other POMs**. Tests compose POMs. Multi-POM orchestration goes in `helpers/`.
- POMs **take `Page` in constructor**. No singletons, no global state.

## Selector Priority

1. `getByRole()` — accessible, resilient to markup changes
2. `getByLabel()`, `getByPlaceholder()`, `getByText()` — user-visible
3. `getByTestId()` — explicit contract
4. `page.locator('[data-*]')` — data attributes from `constants/selectors.ts`
5. CSS selectors — last resort, requires a comment explaining why

**Never use raw CSS selectors when a user-facing locator exists.** If `getByLabel()` fails, it likely
means the app has a real a11y issue (missing `<label for>` association). Fall through the priority
list — e.g. `getByPlaceholder()` — rather than reaching for `page.locator("input[name='...']")`.
Raw CSS selectors couple tests to DOM implementation, not user behavior — the exact fragility trap
that Cypress tests fell into.

**Appsmith-specific:** Many form components (e.g. `FormGroup` from `ads-old`) render visual labels
without proper `<label for>` associations. Prefer `getByPlaceholder()` for these inputs. If neither
label nor placeholder exists, `getByTestId()` is preferred over raw CSS.

## Assertions

Always use Playwright's auto-retrying `expect`:

```typescript
// GOOD — auto-retries until timeout
await expect(locator).toHaveText("Bangladesh");
await expect(locator).toBeVisible();
await expect(locator).toHaveCount(5);

// BAD — resolves once, no retry on failure
expect(await locator.textContent()).toBe("Bangladesh");
expect(await locator.isVisible()).toBe(true);
expect(await locator.count()).toBe(5);
```

## Wait Strategy

Never use hard waits. Always wait for a specific condition:

```typescript
// BAD — hard timeout
await page.waitForTimeout(500);

// BAD — networkidle is unreliable (waits for "no requests for 500ms", flaky and slow)
await page.waitForLoadState("networkidle");

// GOOD — wait for the element that proves the page is ready
await expect(page.locator(SELECTORS.widgetInDeployed("text")).first()).toBeVisible();

// GOOD — wait for API response after a mutation
const response = page.waitForResponse(r => r.url().includes(API.actionsExecute));
await page.getByRole("button", { name: "Update" }).click();
await response;
```

After `page.goto()`, wait for the **first meaningful element** the test cares about — a heading, a table,
a button. Playwright's auto-waiting on assertions handles this naturally. Never use `networkidle` as a
substitute for identifying what "page ready" actually means for your test.

## Test Structure

- One concern per `test()`. If the name has "and" or "&", split it.
- No `let` variables mutated across tests. Use fixtures.
- Setup via API (fast), UI for what you're testing only.
- Test names: behavior-focused, lowercase. `"filters table by country"` not `"2. Validate Widgets"`

## Constants

- API paths, routes, `data-testid` selectors come from `playwright/constants/`. Never hardcode.
- Test data (assertion values like `"Bangladesh"`) stays inline.
- Rule: if the string changes because the **app** changed, it's a constant. If it changes because the **test data** changed, it's inline.

## File Naming

- Specs: `kebab-case.spec.ts`
- POMs: `kebab-case.page.ts` or `kebab-case.component.ts`
- Helpers: `kebab-case.ts`

## Flaky Test Handling

- Never use bare `test.skip()`. Use `test.fixme("reason")` to track why.
- Before merging new specs: `yarn test:pw:flake-check --grep "test name"` (runs 5x).
- If a test is flaky in CI: file a GitHub issue, add `test.fixme("ISSUE-123: description")`.

## Setup & Config Gotchas

- **Auth setup file location:** `auth.setup.ts` lives in `playwright/fixtures/`, not `playwright/tests/`.
  The `setup` project in `playwright.config.ts` must have its own `testDir: "./playwright/fixtures"` —
  otherwise it inherits the top-level `testDir` and never finds the setup file.
- **Bundled Chromium required:** `playwright install chromium` is a one-time prerequisite for local dev.
  The `--ui` mode needs bundled Chromium for its shell — it does not detect Brave, and env vars like
  `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH` don't affect `--ui` mode's browser discovery.
- **Environment variables:** Credentials (`USERNAME`, `PASSWORD`) are loaded from `playwright/.env` via
  `dotenv` in `playwright.config.ts`. The `run-ui.js` script also loads this file so UI mode picks
  them up. Never hardcode credentials in test files.

## Test Scoping — Directory-Based (No Tags)

Scoping is done by **directory structure**, not tags. The folder a test lives in IS its scope.

```
playwright/tests/
  smoke/                     # ~10-15 tests, "is the app alive?" — every push
    login.spec.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tunacosgun/lowcode-studio](https://github.com/tunacosgun/lowcode-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
