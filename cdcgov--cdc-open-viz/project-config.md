---
trigger: always_on
description: This file helps AI coding assistants understand how to work with this repository. Use it as an index to these sections:
---

# AI Agent Instructions

This file helps AI coding assistants understand how to work with this repository. Use it as an index to these sections:

- [Testing Commands](#testing-commands): Required testing strategy plus targeted/quick/full test commands.
- [Test Discovery & Storybook MCP](#test-discovery--storybook-mcp): How AI tools locate tests, story files, and the shared test helpers.
- [Branch Comparison](#branch-comparison): Default base branch for feature diffs.
- [Context Documents](#context-documents): Deep guidance for specific systems and workflows.
- [Config Documentation Maintenance](#config-documentation-maintenance): Rules for keeping package `CONFIG.md` files and shared config docs up to date.

## Branch Comparison

Feature branches usually branch from `dev`; compare against `dev` by default when determining what a feature contains.

## Testing Commands

Use repo-local script commands for consistency (`yarn ...`) and avoid relying on a globally installed `lerna`.

### Required Test Strategy

- Default to targeted tests for the files/components you changed.
- If changes span many areas, run quick suites.
- Do not run full suites unless the user explicitly asks for them.

### Targeted Tests (Default)

For unit targeting, copy this root command shape exactly; both `--` separators are required.

- Unit tests for one package/file:
  - Use package-relative test file paths after `--scope`, not repository-root paths.
  - `yarn test-unit:quick -- --scope @cdc/<package-name> -- src/.../<file>.test.<js|jsx|ts|tsx>`
  - Example: `yarn test-unit:quick -- --scope @cdc/dashboard -- src/test/CdcDashboardComponent.test.tsx`
- Storybook tests for one story file:
  - `yarn test-storybook:quick packages/.../_stories/<file>.stories.<js|jsx|ts|tsx>`
  - Example: `yarn test-storybook:quick packages/chart/src/_stories/ChartEditor.stories.tsx`

### Quick Suites (For Broad Changes)

- Unit quick mode (bypasses standalone build checks by setting `COVE_QUICK_TESTS=1`):
  - `yarn test-unit:quick`
- Storybook quick mode (sets `COVE_QUICK_TESTS=1` and excludes files matching `*.smoke.stories.<js|jsx|ts|tsx>`):
  - `yarn test-storybook:quick`

### Full Suites (Opt-In Only)

Run full suites only when the user explicitly requests them:

- `yarn test-unit`
- `yarn test-storybook`

## Test Discovery & Storybook MCP

The repo is set up so AI tools can discover and run tests without being prompted for file locations or commands.

### Story & test file locations

- Story tests (Storybook `play` functions):
  - `_stories/**/*.stories.@(js|jsx|ts|tsx)`
  - `packages/**/_stories/*.stories.@(js|jsx|ts|tsx)`
- MDX Storybook docs pages:
  - `_stories/**/*.stories.mdx`
  - `packages/**/_stories/*.stories.mdx`
  - These are typically docs-only stories and usually do not contain `play` tests.
- `*.smoke.stories.*` are excluded in quick mode (`COVE_QUICK_TESTS=1`).
- Unit tests: `packages/**/src/**/*.{test,spec}.{js,ts,jsx,tsx}`.
- Test wiring: `vitest.config.ts` and `vitest.setup.ts`.

### Shared test helpers

Story tests should use shared primitives from `@cdc/core/helpers/testing` (`packages/core/helpers/testing.ts`) such as `assertVisualizationRendered`, `performAndAssert`, `waitForEditor`, and `openAccordion`. Prefer these over ad-hoc polling. See `docs/TESTING_BEST_PRACTICES.md`.

### VS Code integrations

- **Tasks** (`.vscode/tasks.json`): `Storybook: Start`, `Test: Storybook (quick)`, `Test: Storybook (target file)`, `Test: Storybook (full)`, `Test: Unit (quick)`, `Test: Unit (full)`. Prefer tasks for execution instead of parsing `package.json`.
- **MCP servers** (`.vscode/mcp.json`): Storybook MCP (`http://localhost:6006/mcp`) and optional Playwright MCP (`stdio`).

### Using the Storybook MCP server

The Storybook MCP server is reachable only while Storybook is running. Run `Storybook: Start` (or `yarn storybook`) first, then use `http://localhost:6006/mcp` to enumerate and inspect stories.

### Using the Playwright MCP server

Playwright MCP is optional and useful for browser-driven debugging/verification from chat. Skip it for code-only changes or routine test execution.

The Playwright MCP package in `.vscode/mcp.json` is pinned for reproducibility. Check for updates frequently with `npm view @playwright/mcp version` (or the `MCP: Check Playwright Latest` task). When a newer stable release is used, update the pin in `.vscode/mcp.json` in the same commit.

## Context Documents

Before working on a specific area, read the relevant context document from the `docs/` directory:

| Document                             | When to Use                                                                                                                                                                      |
| ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `docs/FILTERED_TEXT_DEPRECATION.md`  | Working on the `filtered-text` deprecation. Phase 1 was implemented in `4.26.5`; Phase 2 is planned for `4.26.6`. Covers the two-phase rollout, migration contract, and cleanup/removal expectations. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CDCgov/cdc-open-viz](https://github.com/CDCgov/cdc-open-viz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
