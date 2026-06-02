---
trigger: always_on
description: This is a Playwright end-to-end testing suite for Magento 2 stores running the Hyva theme. It tests frontend flows (login, checkout, cart, account management, etc.) and uses Magento's REST API for setup tasks. Tests run across Chromium, Firefox, and WebKit.
---

# AGENTS.md - AI Agent Guide for magento2-playwright

## Project Overview

This is a Playwright end-to-end testing suite for Magento 2 stores running the Hyva theme. It tests frontend flows (login, checkout, cart, account management, etc.) and uses Magento's REST API for setup tasks. Tests run across Chromium, Firefox, and WebKit.

Also note that this is an open-source project. The tool is downloaded as an npm package, but contributors can actively work on the suite on Github: https://github.com/elgentos/magento2-playwright. Therefore, implementations and improvements to the suite should be as system-agnostic as is possible, and try to make customization as easy as possible.

## Project Structure

```
.
├── base-tests/            # Read-only reference tests (DO NOT MODIFY)
│   ├── config/            # Default JSON config files
│   ├── poms/              # Page Object Models (frontend/ and admin/)
│   ├── utils/             # Utility modules
│   ├── types/             # TypeScript type definitions
│   └── *.spec.ts          # Test specifications
├── tests/                 # Customization layer (EDIT HERE)
│   ├── config/            # Config overrides (deep-merged with base-tests)
│   ├── poms/              # POM overrides
│   ├── utils/             # Utility overrides
│   └── *.spec.ts          # Test overrides and additions
├── .auth/                 # Per-worker auth storage (worker_0.json..worker_5.json)
├── playwright.config.ts   # Playwright configuration
├── tsconfig.json          # Path aliases (@config, @utils/*, @poms/*, etc.)
├── .env                   # Environment variables
├── build.js               # Copies base-tests from node_modules
├── install.js             # Interactive setup wizard
└── .gitlab-ci.yml         # CI/CD pipeline
```

## Base-Tests vs Tests: The Override System

The suite uses a dual-layer architecture. `base-tests/` contains the reference implementation and is rebuilt from the npm package on every install. `tests/` is the customization layer.

**How file resolution works** (in `playwright.config.ts`):
- The `getTestFiles()` function scans both `base-tests/` and `tests/` for `*.spec.ts` files.
- If a file with the same name exists in both directories, only the `tests/` version runs.
- Files unique to `tests/` are included as additional tests.
- Files only in `base-tests/` run as-is.

**Rules for agents:**
- Never modify files in `base-tests/`. They are overwritten on package updates.
- Always make changes in `tests/`. To customize a base test, copy it to `tests/` and modify there.
- The same override logic applies to POMs, utils, and config via TypeScript path aliases.

## Configuration System

Five JSON config files live in `config/`. The loader (`config/index.ts`) deep-merges `base-tests/config/` with `tests/config/`, so you only need to specify overrides in `tests/config/`.

| File | Export | Purpose |
|---|---|---|
| `element-identifiers.json` | `UIReference` | UI element labels, roles, and CSS selectors |
| `input-values.json` | `inputValues` | Test data (names, addresses, credit cards, search terms) |
| `slugs.json` | `slugs` | URL paths for all pages |
| `outcome-markers.json` | `outcomeMarker` | Expected success/error messages |

**Import config like this:**
```typescript
import { UIReference, slugs, outcomeMarker, inputValues } from '@config';
```

## Path Aliases

Defined in `tsconfig.json`. Always use these instead of relative paths:

| Alias | Resolves to |
|---|---|
| `@config` | `base-tests/config` or `tests/config` |
| `@utils/*` | `base-tests/utils/*` or `tests/utils/*` |
| `@poms/*` | `base-tests/poms/*` or `tests/poms/*` |
| `@types/*` | `base-tests/types/*` or `tests/types/*` |
| `@fixtures/*` | `base-tests/fixtures/*` or `tests/fixtures/*` |

## Authentication & Fixtures

Tests that require a logged-in user import `test` from `@utils/fixtures.utils` instead of `@playwright/test`:

```typescript
import { test } from '@utils/fixtures.utils';
```

This custom fixture:
- Assigns each Playwright worker a unique account (`playwright_user_{id}@elgentos.nl`).
- Stores auth state in `.auth/worker_{id}.json`.
- Logs in once per worker (not per test) and reuses the session.
- Validates existing sessions before reusing them.

Tests that don't need authentication import directly from `@playwright/test`:
```typescript
import { test, expect } from '@playwright/test';
```

## Page Object Model Pattern

POMs live in `poms/frontend/` and `poms/admin/`. Each POM:
- Takes a `Page` in its constructor.
- Defines locators as `readonly` properties using config values (never hardcoded strings).
- Exposes action methods (e.g., `login()`, `addToCart()`).
- Uses `UIReference` for element labels and `slugs` for navigation.

Example:
```typescript
import { UIReference, slugs } from '@config';

class LoginPage {
  readonly page: Page;
  readonly loginEmailField: Locator;

  constructor(page: Page) {
    this.page = page;
    this.loginEmailField = page.getByRole('textbox', {
      name: UIReference.credentials.emailFieldLabel, exact: true
    });
  }


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elgentos/magento2-playwright](https://github.com/elgentos/magento2-playwright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
