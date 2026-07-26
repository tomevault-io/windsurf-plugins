---
trigger: always_on
description: Playwright Azure Reporter is a TypeScript package that creates a Playwright reporter for integrating test results with Azure DevOps Test Plans. It allows publishing test results directly to Azure DevOps using test case IDs embedded in test names or annotations.
---

# Playwright Azure Reporter

Playwright Azure Reporter is a TypeScript package that creates a Playwright reporter for integrating test results with Azure DevOps Test Plans. It allows publishing test results directly to Azure DevOps using test case IDs embedded in test names or annotations.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## CRITICAL: Timeout Requirements

**NEVER CANCEL LONG-RUNNING COMMANDS**. This repository has specific timing requirements:

- **Dependencies**: `yarn install` - 60+ second timeout required  
- **Build**: `yarn build` - 30+ second timeout required
- **Tests**: `yarn test:reporter` - 120+ second timeout required (59 tests)
- **Complete workflow**: 180+ second timeout required for full validation
- **Browser installation**: Will fail due to network issues - document but continue

**Set explicit timeouts for ALL commands to prevent premature cancellation.**

## Working Effectively

- **Bootstrap the repository:**
  - `yarn install --ignore-engines` - Installs all dependencies. Takes ~15 seconds. NEVER CANCEL. Set timeout to 60+ seconds.
  - `npx playwright install` - Installs Playwright browsers. May fail due to network issues - document if it fails.

- **Build the project:**
  - `yarn build` - Full build (clean + lint + TypeScript compile). Takes ~8 seconds. NEVER CANCEL. Set timeout to 30+ seconds.
  - `yarn clean` - Removes dist/ directory. Takes <1 second.
  - `yarn dev` - Development build (lint + TypeScript compile). Takes ~8 seconds. Set timeout to 30+ seconds.

- **Run tests:**
  - `yarn test:reporter` - Run reporter unit tests. Takes ~50 seconds (59 tests). NEVER CANCEL. Set timeout to 120+ seconds.
  - `yarn test` - Run example integration tests. REQUIRES Playwright browsers to be installed.

- **Code quality:**
  - `yarn lint` - Type check and ESLint validation. Takes ~3 seconds. Set timeout to 30+ seconds.
  - `yarn lint:fix` - Auto-fix ESLint issues. Takes ~2 seconds. Set timeout to 30+ seconds.
  - `yarn format` - Format code with Prettier. Takes ~1.5 seconds. Set timeout to 30+ seconds.

## Validation

- **ALWAYS run the reporter test suite after making changes:**
  - `yarn test:reporter` validates the core reporter functionality without requiring browser installation
  - These tests cover all reporter modes, configuration validation, and Azure DevOps integration logic
  - Set timeout to 120+ seconds as tests take ~50 seconds to complete

- **Test the example suite only if browsers are available:**
  - The main test suite (`yarn test`) requires Playwright browsers and demonstrates Azure DevOps test case ID formats
  - If browsers fail to install due to network issues, document this but don't let it block your work

- **ALWAYS run code quality checks before completing changes:**
  - `yarn lint` - Type checking and linting (required for CI). Takes ~3 seconds. Set timeout to 30+ seconds.
  - `yarn format` - Code formatting (required for CI). Takes ~1.5 seconds. Set timeout to 30+ seconds.
  - These are enforced by the pre-commit hook and CI pipeline

- **Complete development workflow validation:**
  - Full workflow: `yarn build && yarn test:reporter && yarn lint && yarn format`
  - Takes ~63 seconds total. NEVER CANCEL. Set timeout to 180+ seconds.

## Playwright Browser Installation Issues

- **KNOWN ISSUE**: `npx playwright install` consistently fails with network timeouts or download failures
- **Error signature**: "Download failed: size mismatch" or "Download failure, code=1"
- **Impact**: Only affects the example integration tests (`yarn test`), not core development
- **Workaround**: Continue development without browsers - the reporter tests cover all functionality
- **Resolution**: Document the failure but do not let it block development work

## Common Tasks

### Building and Testing Workflow
```bash
# Complete development workflow (takes ~63 seconds total)
yarn install --ignore-engines     # ~15 seconds, timeout: 60+ seconds
yarn build                        # ~8 seconds, timeout: 30+ seconds  
yarn test:reporter                 # ~50 seconds, timeout: 120+ seconds
yarn lint                         # ~3 seconds, timeout: 30+ seconds
yarn format                       # ~1.5 seconds, timeout: 30+ seconds
```

### Code Quality Validation
```bash
# Before committing (enforced by pre-commit hook)
yarn lint          # Type checking + ESLint - ~3 seconds
yarn lint:fix      # Auto-fix linting issues - ~2 seconds
yarn format        # Format with Prettier - ~1.5 seconds
```

### Development Mode
```bash
# Watch mode requires nodemon to be installed globally
yarn dev:watch     # Auto-rebuild on file changes (if nodemon available)
# Alternative: Run yarn dev manually after changes
```

## Repository Structure

### Source Code (`src/`)
- `playwright-azure-reporter.ts` - Main reporter implementation
- `logger.ts` - Logging utilities
- `utils.ts` - Helper functions

### Tests (`tests/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexneo2003/playwright-azure-reporter](https://github.com/alexneo2003/playwright-azure-reporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
