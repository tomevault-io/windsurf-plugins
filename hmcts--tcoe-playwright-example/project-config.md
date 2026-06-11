---
trigger: always_on
description: This is an **HMCTS Playwright test automation template** following **Page Object Model (POM)** with hierarchical component structure:
---

# Copilot Instructions for HMCTS Playwright Template

## Architecture Overview

This is an **HMCTS Playwright test automation template** following **Page Object Model (POM)** with hierarchical component structure:
- **Base class** (`playwright-e2e/page-objects/base.ts`) - Provides common components to all pages via composition
- **Components** (`playwright-e2e/page-objects/components/`) - Reusable UI sections (headers, case lists) shared across pages
- **Pages** (`playwright-e2e/page-objects/pages/`) - Full page implementations extending `Base`
- **Fixtures** (`playwright-e2e/fixtures.ts`) - Dependency injection system for pages and utilities
- **Utils** (`playwright-e2e/utils/`) - Focused utility classes (validators, config, cookie handlers)

## Key Patterns & Conventions

### Test Organization
- Tests tagged for selective execution: `@a11y`, `@performance`, `@visual`, `@smoke`, `@exui`, `@cui`
- Global setup/teardown (`global.setup.ts`, `global.teardown.ts`) handles authentication and session management
- Performance tests require `@performance` tag AND `lighthousePage` fixture (chromium only)
- **Always import from centralized fixtures**: `import { test, expect } from "../fixtures"` (never from `@playwright/test`)

### Fixture-Based Dependency Injection
```typescript
// Fixtures provide pre-configured page objects and utilities
test("example", async ({ exuiCaseListPage, axeUtils, config, idamPage }) => {
  // All dependencies injected automatically - no manual instantiation
  await exuiCaseListPage.goto();
  await axeUtils.audit();
});
```

### User & Session Management
- Global setup creates sessions for user roles (caseManager, judge) stored in `.sessions/`
- Use `test.use({ storageState: config.users.caseManager.sessionFile })` to select session
- Dynamic citizen users: `await citizenUserUtils.createUser()` returns ephemeral credentials
- IDAM tokens via `idamUtils.generateIdamToken()` for API operations
- Session validation: `SessionUtils.isSessionValid(sessionFile, cookieName)` avoids re-authentication

### Page Object Model Structure
```typescript
// All pages extend Base to inherit common components
export class ExuiCaseListPage extends Base {
  readonly container = this.page.locator("exui-case-home");
  
  constructor(page: Page) {
    super(page); // Provides exuiHeader, exuiCaseListComponent, etc.
  }
  
  async goto() {
    await this.page.goto(config.urls.manageCaseBaseUrl);
    await this.exuiHeader.checkIsVisible();
  }
}
```

## Essential Commands

### Test Execution
```bash
# Browser-specific (excludes a11y, performance, visual by default)
yarn test:chrome
yarn test:firefox  
yarn test:webkit
yarn test:edge
yarn test:tabletchrome
yarn test:tabletwebkit

# Specialized test types
yarn test:a11y              # Accessibility tests with axe-core (chrome only)
yarn test:visual            # Visual regression with snapshots (chromium)
yarn test:update-snapshots  # Update visual baselines
yarn playwright test --grep @performance --project=chromium  # Performance tests
```

### Development & CI
```bash
yarn lint                    # TypeScript + ESLint validation (CI enforced)
yarn setup                   # Install Playwright browsers with dependencies
yarn setup:edge              # Install Edge browser separately
yarn start-container         # Run tests in Docker (reproduces CI environment)
```

## Critical Dependencies

- **`@hmcts/playwright-common`** - HMCTS shared components (`ExuiCaseListComponent`, `IdamPage`, `LighthouseUtils`, `AxeUtils`, `SessionUtils`, `TableUtils`, `BrowserUtils`)
- **Session storage** - `.sessions/` directory contains browser state (cookies) for authenticated users
- **Lighthouse** - Performance testing (chromium-based browsers only)
- **Axe-core** - Accessibility testing via `@axe-core/playwright`

## Configuration Patterns

- **Config centralization**: `playwright-e2e/utils/config.utils.ts` handles all environment variables
- **Environment variables**: Use `getEnvVar(name, fallback?)` for optional, `requireEnvVar(name)` for required
- **Never hardcode**: URLs, credentials, secrets - always use `process.env` or config
- **Browser projects**: Defined in `playwright.config.ts` with `dependencies: ["setup"]` ensuring global setup runs first
- **Test data isolation**: Each test uses unique data to prevent conflicts

## Development Practices & HMCTS Standards

> 👉 **First steps**: Familiarise yourself with `docs/BEST_PRACTICE.md` (HMCTS Playwright coding standard) and `docs/CONFIGURATION.md`. Everything below builds on those documents.

### SOLID Principles Implementation

**Single Responsibility Principle (SRP)**
- **Pages**: Handle navigation and expose components only (e.g., `ExuiCaseListPage.goto()`)
- **Components**: Manage specific UI sections (e.g., `ExuiHeaderComponent.checkIsVisible()`)
- **Utils**: Focused single-purpose classes (e.g., `ValidatorUtils` validates formats, `CookieUtils` manages cookies)
- **Anti-pattern**: Avoid multi-purpose classes doing "X and Y" - split them

**Open/Closed Principle (OCP)**
- `Base` class provides extension via composition (components added, not modified)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hmcts/tcoe-playwright-example](https://github.com/hmcts/tcoe-playwright-example) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
