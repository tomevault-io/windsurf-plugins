---
trigger: always_on
description: This repository uses Nx as the task runner and monorepo manager for building and testing Angular ESLint packages.
---

# Guidelines for Claude Code

This repository uses Nx as the task runner and monorepo manager for building and testing Angular ESLint packages.

## Package Manager

This project uses **pnpm** (version 10) as the package manager. Always use `pnpm` commands instead of `npm` or `yarn`.

## Required checks

When modifying rule implementations or documentation, run the following commands and ensure they pass (noting the comments that explain what to do if any of these checks fail):

```bash
pnpm format-check # run pnpm format and commit the result if this check fails
pnpm nx sync:check # run pnpm nx sync and commit the result if this check fails
pnpm nx run-many -t check-rule-docs # run pnpm nx run-many -t update-rule-docs and commit the result if this check fails
pnpm nx run-many -t check-rule-lists # run pnpm nx run-many -t update-rule-lists and commit the result if this check fails
pnpm nx run-many -t check-rule-configs # run pnpm nx run-many -t update-rule-configs and commit the result if this check fails
```

When working on an individual rule, the preferred way to run tests is to target the specific spec file. For example, to run tests for the `prefer-standalone` rule within the `eslint-plugin` project, run:

```bash
pnpm nx test eslint-plugin packages/eslint-plugin/tests/rules/prefer-standalone/spec.ts
```

Once rule specific tests have passed, run commands for all projects:

Use `pnpm nx run-many -t test --parallel 2` to run all tests across all packages.
Use `pnpm nx run-many -t lint --parallel 2` to run all linting across all packages.
Use `pnpm nx run-many -t typecheck --parallel 2` to run TypeScript type checking across all packages.

If you are updating e2e tests, you may need to update the snapshots. We use Nx's atomizer feature so that each e2e test file gets its own dynamic target based on the file, so for example if you need to update `src/inline-template-fixer.test.ts`, you can try running `pnpm nx run e2e:e2e-ci--src/inline-template-fixer.test.ts -u` to update the snapshots. If that is unsuccessful e.g. for npm auth reasons, you can try running `npx nx run e2e:e2e-local --configuration updateSnapshot` to update all the e2e snapshots. NOTE: `npx` should be used here, not `pnpm`, so that the verdaccio local registry auth works correctly.

You should commit the resulting snapshot changes. NOTHING ELSE. There will be a diff on package.json files etc when doing this, but ONLY commit the snapshot changes.

## Project Structure

This is a monorepo with the following main packages in `/packages/`:

- **angular-eslint**: Main package that provides configurations
- **eslint-plugin**: Core ESLint rules for Angular TypeScript code
- **eslint-plugin-template**: ESLint rules for Angular HTML templates
- **builder**: Angular CLI builder for running ESLint
- **schematics**: Angular schematics for adding ESLint to projects
- **template-parser**: Parser for Angular templates
- **test-utils**: Testing utilities
- **utils**: Shared utility functions
- **bundled-angular-compiler**: Bundled Angular compiler
- **nx-plugin**: Nx plugin for the local Angular ESLint monorepo, not published to npm

## Code Conventions

### TypeScript Configuration

- **Strict TypeScript**: Uses strict mode with additional strict flags like `noImplicitReturns`, `noUnusedLocals`, `noFallthroughCasesInSwitch`
- **Target**: ES2022 with Node.js module resolution
- **Module system**: NodeNext for modern Node.js compatibility

### ESLint Configuration

- Uses **flat config** format (eslint.config.js)
- Nx ESLint plugin for monorepo management
- JSONC parser for JSON files
- Enforces module boundaries between packages

### File Naming and Structure

- **Test files**: Use `.spec.ts` suffix and live in `tests/` directories
- **Rule files**: Individual TypeScript files in `src/rules/`
- **Config files**: JSON files in `src/configs/`
- **Documentation**: Markdown files in `docs/rules/` matching rule names. NOTE: THESE ARE AUTO-GENERATED, DO NOT EDIT THEM MANUALLY.
- **Test cases**: Organized in `tests/rules/{rule-name}/cases.ts` and `tests/rules/{rule-name}/spec.ts`

### Rule Development Patterns

When creating or modifying ESLint rules:

1. Rules are implemented in TypeScript using the `@typescript-eslint/utils` package
2. Each rule has its own file in `src/rules/`
3. Rule documentation is auto-generated and lives in `docs/rules/`
4. Test cases follow a pattern with `cases.ts` and `spec.ts` files
5. Rules use a utility function `createESLintRule()` for consistent structure

### Testing Conventions

- **Vitest**: Primary testing framework
- **Rule testing**: Uses `@typescript-eslint/rule-tester` for ESLint rule testing
- **Snapshot testing**: Used for E2E tests and some rule outputs
- **Test organization**: Tests mirror the source structure in separate `tests/` directories

### Build and Release

- **Build**: Uses Nx with TypeScript compilation
- **Targets**: Multiple build targets (`build`, `compile`) with dependency management
- **Release**: Conventional Commits with automated changelogs
- **Versioning**: Uses Nx release management with GitHub integration

Claude should NEVER run versioning or publishing commands.

## Commit conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [angular-eslint/angular-eslint](https://github.com/angular-eslint/angular-eslint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
