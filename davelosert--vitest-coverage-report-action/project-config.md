---
trigger: always_on
description: This is a GitHub Action that reports Vitest test coverage as GitHub step summaries and pull request comments. The action parses Vitest JSON coverage reports, generates HTML summaries, and posts them to PRs or commits.
---

# GitHub Copilot Instructions

## Project Overview

This is a GitHub Action that reports Vitest test coverage as GitHub step summaries and pull request comments. The action parses Vitest JSON coverage reports, generates HTML summaries, and posts them to PRs or commits.

## Tech Stack

### Core Technologies

- **TypeScript**: Compiled with `target: ES2019`, `module: commonjs`, strict mode enabled
- **Node.js 24**: Runtime environment for GitHub Actions
- **esbuild**: Bundles TypeScript to a single `dist/index.js` file for deployment
- **Vitest**: Testing framework with coverage support via `@vitest/coverage-v8`
- **Biome**: Linter and formatter (replaces ESLint/Prettier)

### Key Dependencies

- `@actions/core`: GitHub Actions toolkit for inputs/outputs/logging
- `@actions/github`: GitHub API integration via Octokit
- `common-tags`: Template literal tag library for HTML generation

## Project Architecture

### Key Design Patterns

1. **Functional Organization**: Code organized by feature domain (inputs, report generation, output)
2. **Separation of Concerns**: Input parsing, report generation, and output writing are isolated
3. **Type Safety**: Strong typing with TypeScript interfaces for all data structures
4. **Error Handling**: Explicit handling of GitHub API RequestErrors (403, 404, 422)
5. **Test Factories**: Mock factories in `types/` for consistent test data generation

## Code Style & Conventions

### Language Features
- **Prefer `for...of` loops** over `forEach` statements
- **Use template strings** over string concatenation
- **Use `.js` extensions** in import statements (ESM-style imports even though compiled to CommonJS)
- **Prefer `async/await`** over promise chains
- **Use type imports**: `import type { ... }` for type-only imports

### Code Organization
- **Co-locate tests**: Test files live alongside implementation (e.g., `options.ts` + `options.test.ts`)
- **No barrel exports**: Import directly from specific files
- **Descriptive function names**: Use clear, verb-based names (e.g., `generateFileCoverageHtml`, `getPullChanges`)

### TypeScript
- Use `type` for object shapes and unions
- Use `interface` sparingly (project prefers `type`)
- Enable all strict compiler options
- Use `satisfies` operator when appropriate for type narrowing

## Testing Standards

### Framework & Configuration
- **Framework**: Vitest with coverage via `@vitest/coverage-v8`
- **Test Pattern**: `**/*.test.ts` files
- **Coverage Requirements**:
  - Reporters: `json-summary` and `json` (required)
  - Include: `src/` directory
  - Exclude: `src/types/` and `**/*.test.ts`
  - All files coverage tracking enabled

### Test Requirements
- **Tests MUST pass** before any code changes are committed
- **Coverage MUST be maintained** or improved
- Use `describe()` blocks to group related tests
- Use `it()` or `test()` for individual test cases
- Mock external dependencies (GitHub API, file system)
- Use factories from `types/*MockFactory.ts` for test data

### Running Tests
```bash
npm test              # Run tests in watch mode
npm run test:coverage # Generate coverage report
```

## Linting & Formatting

### Biome Configuration
- **Linter**: Enabled with recommended rules
- **Files**: Lints `src/`, `test/`, and `biome.json`
- **Organize imports**: Automatically enabled
- **No configuration extensions**: Uses Biome defaults

### Lint Requirements
- **Lint MUST pass** before committing code
- **No lint warnings** should be introduced
- Run `npm run lint` to check
- Run `npm run lint:write` to auto-fix issues
- CI uses `npm run lint:ci` for strict checking

### Commands
```bash
npm run lint        # Check for issues
npm run lint:write  # Fix auto-fixable issues
npm run lint:ci     # CI mode (strict)
```

## Build & Deployment

### Build Process
1. TypeScript compiled via esbuild
2. Bundled into single `dist/index.js`
3. Minified with sourcemaps
4. Target: Node 16+ (GitHub Actions compatibility)

### Build Commands

```bash
npm run build     # Clean and build
npm run typecheck # Type-check without emitting
npm start         # Build and run locally
```

### GitHub Action Deployment
- Entry point: `dist/index.js`
- Runtime: `node20`
- Action definition: `action.yml`

## Development Workflow

### Before Committing

1. **Run tests**: `npm test`
2. **Check lint**: `npm run lint`
3. **Type check**: `npm run typecheck`
4. **Build**: `npm run build`

### Adding New Features

1. Determine the appropriate directory (`inputs/`, `report/`, root)
2. Create implementation file with `.ts` extension
3. Create co-located test file with `.test.ts` extension
4. Write tests using Vitest's `describe()` and `it()`
5. Ensure tests pass and coverage is maintained
6. Run lint to ensure code style compliance
7. Update types in `types/` directory if needed

### Modifying Existing Code

1. Locate the relevant file in the appropriate directory
2. Make changes while maintaining existing patterns
3. Update co-located tests
4. Run `npm test` to verify changes
5. Run `npm run lint:write` to apply formatting
6. Ensure `npm run typecheck` passes

## GitHub Actions Integration

### Input Handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davelosert/vitest-coverage-report-action](https://github.com/davelosert/vitest-coverage-report-action) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
