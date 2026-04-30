---
trigger: always_on
description: This document provides guidelines for developers and AI agents contributing to Devvami.
---

# Contributing with AI: Development Guidelines

This document provides guidelines for developers and AI agents contributing to Devvami.

## Project Overview

**Devvami** is a DevEx CLI (Developer Experience) tool written in JavaScript (ESM) that helps teams manage GitHub repositories, CI/CD pipelines, AWS costs, and task management from the terminal.

- **Language**: JavaScript (ESM, `.js` files)
- **Runtime**: Node.js >= 24
- **CLI Framework**: @oclif/core v4
- **Package Manager**: pnpm >= 10

## Repository Structure

```
devvami/
├── src/
│   ├── commands/          # CLI commands (oclif auto-discovers)
│   │   ├── auth/
│   │   ├── costs/
│   │   ├── create/
│   │   ├── docs/
│   │   ├── pipeline/
│   │   ├── pr/
│   │   ├── repo/
│   │   ├── tasks/
│   │   └── [root commands]
│   ├── services/          # Business logic (GitHub, AWS, ClickUp, etc.)
│   ├── formatters/        # Output formatting (tables, markdown, etc.)
│   ├── hooks/             # oclif hooks (init, postrun)
│   ├── utils/             # Utilities (errors, banner, colors, etc.)
│   ├── validators/        # Input validation
│   ├── types.js           # JSDoc type definitions
│   ├── help.js            # Custom help system
│   └── index.js           # Main entry point
├── tests/
│   ├── unit/              # Unit tests
│   ├── services/          # Service/integration tests
│   ├── integration/       # End-to-end CLI tests
│   ├── fixtures/          # Mock data and tools
│   └── setup.js           # Test configuration
├── bin/                   # CLI entry points
├── .github/workflows/     # GitHub Actions CI/CD
├── package.json
├── vitest.config.js
└── lefthook.yml           # Git hooks (lint, test, commitlint)
```

## Code Style

### JavaScript Standards

- **ESM** (ECMAScript Modules) only, no CommonJS
- **JSDoc** required on all public functions
- **Type annotations** in JSDoc comments
- **No TypeScript** — pure JavaScript with JSDoc

### JSDoc Requirements

Every public function/export must have JSDoc with `@param`, `@returns`, and `@typedef` for types:

```javascript
/**
 * Fetch pull requests for a repository.
 * @param {string} org - GitHub organization name
 * @param {string} repo - Repository name
 * @param {Object} options - Options
 * @param {number} [options.limit=10] - Max results
 * @returns {Promise<Array<{number: number, title: string}>>}
 */
export async function getPullRequests(org, repo, options = {}) {
  // ...
}
```

### Code Organization

- **One command = one file** in `src/commands/`
- **Services handle API calls** — never in commands
- **Formatters handle output** — use chalk for colors, ora for spinners
- **Utils for cross-cutting concerns** — errors, logging, helpers
- **Config is loaded from `~/.config/dvmi/config.json`** — never hardcode

### Naming Conventions

- **Files**: lowercase with hyphens (`pull-requests.js`, not `pullRequests.js`)
- **Commands**: match directory structure (`src/commands/pr/create.js` → `dvmi pr create`)
- **Functions**: camelCase
- **Constants**: UPPER_SNAKE_CASE
- **Classes**: PascalCase

### Error Handling

All errors should extend `DvmiError` from `src/utils/errors.js`:

```javascript
import { DvmiError } from '../utils/errors.js'

if (!config.org) {
  throw new DvmiError(
    'No organization configured',
    'Run `dvmi init` to set up your organization'
  )
}
```

## Testing

Tests use **Vitest** with the following structure:

- **Unit tests**: `tests/unit/` — test pure functions
- **Service tests**: `tests/services/` — test API integration logic
- **Integration tests**: `tests/integration/` — test full CLI flows

### Running Tests

```bash
pnpm test                 # Run all tests
pnpm test:unit           # Run unit tests only
pnpm test:services       # Run service tests only
pnpm test:integration    # Run integration tests only
pnpm test:watch          # Watch mode
pnpm test:coverage       # Generate coverage report
```

### Test Standards

- **Test files**: `*.test.js` co-located near source
- **Fixtures**: `tests/fixtures/` for mock data, stub binaries
- **MSW**: Mock Service Worker for HTTP mocking
- **Coverage target**: 80%+ (enforced by pre-commit hooks)

Example:

```javascript
import { describe, it, expect, beforeEach } from 'vitest'
import { getPullRequests } from '../src/services/github.js'

describe('GitHub Service', () => {
  it('fetches PRs for a repository', async () => {
    const prs = await getPullRequests('acme', 'my-app')
    expect(prs).toHaveLength(1)
    expect(prs[0].title).toBe('Add cool feature')
  })
})
```

## Linting & Formatting

- **ESLint**: Enforces code style, JSDoc validation
- **Prettier**: Auto-formats code
- **Commitlint**: Validates commit messages
- **Lefthook**: Runs checks on pre-commit, pre-push

### Commands

```bash
pnpm lint         # Run ESLint
pnpm lint:fix     # Fix auto-fixable issues
pnpm format       # Format with Prettier
pnpm commit       # Interactive commit (uses commitlint)
```

## Commit Conventions

Follow **Conventional Commits**:

```
<type>(<scope>): <subject>

<body>
<footer>
```

**Types**:
- `feat` — New feature
- `fix` — Bug fix
- `docs` — Documentation
- `style` — Code style (no behavior change)
- `refactor` — Code refactor (no behavior change)
- `perf` — Performance improvement

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [savez/devvami](https://github.com/savez/devvami) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
