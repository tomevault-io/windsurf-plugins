---
trigger: always_on
description: Git Truck is a tool for visualizing Git repositories. It consists of a CLI interface that spins up a full-stack web application.
---

# Git Truck

## Project Overview

Git Truck is a tool for visualizing Git repositories. It consists of a CLI interface that spins up a full-stack web application.

## Tech Stack

- **Runtime**: Bun
- **Framework**: React Router v7 (Framework Mode)
- **Styling**: Tailwind CSS
- **Language**: TypeScript
- **Visualization layouts**: D3.js
- **Database**: DuckDB (via `@duckdbw/node-api`)
- **Testing**: Vitest (Unit), Playwright (E2E)

## Architecture

- **CLI Entry**: `src/cli.ts` - Entry point for the CLI.
- **Server Entry**: `src/server/app.ts` - Main server application logic.
- **App Directory**: `src/` - Contains the React Router application.
  - `routes/`: Application routes.
  - `analyzer/`: Core logic for analyzing Git repositories.
  - `components/`: React components.
  - `metrics/`: Metrics calculation logic.

## Key Commands

- **Install Dependencies**: `bun install`
- **Start Dev Server**: `bun dev` or `bun dev:bun`
- **Build**: `bun run build`
- **Run Unit Tests**: `bun test:unit`
- **Run E2E Tests**: `bun test:e2e`
- **Lint & Format**: `bun check`
- **Typecheck**: `bun typecheck`

## TypeScript Guidelines

- **Never use `any`**: Avoid the `any` type in all TypeScript code. Always use specific types or generics. Refactor code to eliminate any usage whenever possible.
- **Prefer `type` over `interface`**: Use `type` aliases for defining shapes and unions. Only use `interface` when extending classes or for compatibility with libraries that require it.

## Development Guidelines

- **Task Execution**: Prefer using VS Code tasks (via `run_task`) for common operations like building, testing, or starting the server. Fall back to `bun` CLI commands in the terminal if no suitable task exists.
- **Package Manager**: Use `bun` for all script execution and package management.
- **Dependency Placement**: Keep published runtime dependencies as small as possible. The package is built as bundled artifacts: Vite bundles the React Router app/server build, and tsdown bundles the CLI. Dependencies that are only needed to build those artifacts should live in `devDependencies`, even if app or CLI source imports them at runtime before bundling.
- **Runtime Dependency Exceptions**: Keep packages in `dependencies` only when the published `build/` output or `cli.mjs` intentionally loads them as external packages. DuckDB (`@duckdb/node-api` and its platform bindings) is intentionally external because it ships system-specific binaries and must remain a production dependency.
- **Bundling Config**: When adding a package used by the CLI, make sure tsdown bundles it by default or add it to `tsdown.deps.alwaysBundle`. Do not add packages to `tsdown.deps.neverBundle` unless they truly must be installed by users at runtime.
- **Dependency Verification**: After changing dependencies or bundling config, run `bun run build` and scan the published artifacts for bare package imports. Only Node built-ins and intentional externals such as `@duckdb/node-api` should appear:
  `rg -n "^import .* from \"[^./]|^import \"[^./]" cli.mjs build/server -g '!node_modules'`
- **Commit Messages**: Use Conventional Commits in the form `type: subject`. Keep the subject imperative, under 50 characters, with no trailing period. Prefer the most semantic type first; use `chore:` only when the change does not fit any of the categories below. Use these types in this order:
  - `feat:` - A new feature
  - `fix:` - A bug fix
  - `docs:` - Documentation only changes
  - `style:` - Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc)
  - `refactor:` - A code change that neither fixes a bug nor adds a feature
  - `perf:` - A code change that improves performance
  - `test:` - Adding missing tests or correcting existing tests
  - `build:` - Changes that affect the build system or external dependencies (example scopes: gulp, broccoli, npm)
  - `ci:` - Changes to CI configuration files and scripts (example scopes: Travis, Circle, BrowserStack, SauceLabs)
  - `chore:` - Other changes that do not fit the categories above
  - `revert:` - Reverts a previous commit
- **Release Workflow**: Use Changesets for releases. Add a changeset for releasable work, let `.github/workflows/release.yml` open or update the release PR on `main`, and expect npm publishing to happen automatically after that release PR is merged.
- **Styling**: Use Tailwind CSS utility classes.
- **Routing**: Follow React Router file-based routing and loaders/actions conventions in `src/routes/`.
- **Database**: Interactions often involve DuckDB or direct git command execution.
- **Database schema changes**: Do not patch old DuckDB databases with `ALTER TABLE` migrations that add missing analysis columns with placeholder values. If existing data cannot be made semantically complete, rely on a major version bump to trigger the automatic full re-analysis instead.
- **Imports**: Use the path alias `~/` for imports from the `src/` directory instead of relative imports (e.g., `import { foo } from "~/components/Bar"` instead of `import { foo } from "../../../components/Bar"`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [git-truck/git-truck](https://github.com/git-truck/git-truck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
