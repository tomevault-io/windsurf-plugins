---
trigger: always_on
description: - This file applies to the repository root: `/home/jdide/path/siigo-mcp`.
---

# AGENTS.md

## Scope

- This file applies to the repository root: `/home/jdide/path/siigo-mcp`.
- The project is a Node 18+ TypeScript MCP server for the Siigo API.
- Runtime entrypoint: `src/index.ts`.
- Main client implementation: `src/siigo-client.ts`.
- Shared API contracts: `src/types.ts`.
- Tests currently live in `test/*.test.js` and run against built output in `dist/`.

## Agent Priorities

- Keep changes small and targeted.
- Preserve the existing public API shape unless the task explicitly requires a breaking change.
- Prefer extending existing patterns over introducing new abstractions.
- Do not add new tooling unless the task calls for it.
- Do not invent lint or formatter rules that are not already implied by the codebase.

## Repository Facts

- Package manager: `pnpm` (`packageManager: pnpm@10.32.0`).
- Module system: ESM (`"type": "module"`).
- TypeScript config is strict (`"strict": true`).
- Local source imports use explicit `.js` extensions because TS compiles to Node ESM.
- Jest is configured through `jest.config.cjs` with `testEnvironment: 'node'`.
- Jest matches `**/test/**/*.test.js`.
- Tests import from `../dist/...`, so the project must be built before tests are reliable.
- Linting and formatting are handled by Biome through `biome.json`.
- Repo lint script: `pnpm lint`.
- Repo format scripts: `pnpm format` and `pnpm format:write`.
- No Prettier or ESLint config files exist today.
- No Cursor rules were found in `.cursor/rules/` or `.cursorrules`.
- No Copilot instructions were found in `.github/copilot-instructions.md`.

## Setup

- Install dependencies: `pnpm install`
- Build once before running tests: `pnpm build`
- Start compiled server: `pnpm start`
- Run dev entrypoint with TS directly: `pnpm dev`

## Build, Test, and Verification Commands

- Build: `pnpm build`
- Lint: `pnpm lint`
- Auto-fix lint and formatting issues where safe: `pnpm lint:fix`
- Check formatting only: `pnpm format`
- Rewrite files with configured formatting: `pnpm format:write`
- Clean build output: `pnpm clean`
- Full test suite: `pnpm test`
- Start compiled server: `pnpm start`
- Run the TypeScript entrypoint directly: `pnpm dev`

## Single-Test Commands

- Run one test file after building:
  - `pnpm build && node --experimental-vm-modules ./node_modules/jest/bin/jest.js test/siigo-client.test.js`
- Run one named test after building:
  - `pnpm build && node --experimental-vm-modules ./node_modules/jest/bin/jest.js test/siigo-client.test.js -t "searchProducts scans later pages before filtering partial matches"`
- Run tests matching a pattern after building:
  - `pnpm build && node --experimental-vm-modules ./node_modules/jest/bin/jest.js -t "SiigoApiError"`

## Important Verification Notes

- `pnpm test` already builds first.
- Because tests import from `dist/`, source-only edits are not enough; rebuild before asserting test results.
- In the current checkout, `pnpm build` fails if dependencies are not installed because `tsc` is missing from `node_modules`.
- If `node_modules` is absent, run `pnpm install` before any build or test command.
- `pnpm lint` runs Biome across `src`, `test`, and key config files.
- `pnpm lint:fix` applies safe Biome fixes and formatting changes.

## Code Organization

- `src/index.ts` wires environment config, MCP server creation, tool registration, and tool-level error formatting.
- `src/siigo-client.ts` owns HTTP behavior, authentication, pagination helpers, endpoint methods, and API error translation.
- `src/types.ts` holds the shared TypeScript model layer for Siigo request and response shapes.
- `test/siigo-client.test.js` uses Jest ESM mocks and tests the compiled client API.

## Style Guidelines

## Formatting

- Use 2-space indentation.
- Use semicolons.
- Use single quotes.
- Biome is configured to enforce the shared formatting baseline.
- Keep trailing commas where the surrounding code already uses them.
- Prefer multiline object literals and function calls when argument lists are long.
- Follow the existing section-divider comment style when touching large files, but do not add decorative comments unnecessarily.

## Imports

- Group external imports before local imports.
- Keep local imports relative.
- Use explicit `.js` extensions in local imports from TypeScript source, for example `./siigo-client.js`.
- Import TypeScript types using the existing style in the file; this repo currently mixes value and type imports in one statement and does not require `import type`.
- Avoid introducing barrel files unless there is a clear repository-wide need.

## Types

- Maintain strict TypeScript compatibility.
- Prefer explicit interfaces and type aliases for API contracts.
- Use `unknown` instead of `any` when the value is not yet narrowed.
- Narrow errors with runtime checks such as `error instanceof Error` or `axios.isAxiosError(error)`.
- Use `Partial<T>` for partial update payloads when that matches the endpoint behavior.
- Use `Record<string, unknown>` for flexible object payloads instead of loose `object` typing.
- Preserve snake_case fields when they mirror the external Siigo API schema.
- Use camelCase for internal variables, helpers, methods, and config properties.

## Naming Conventions

- Classes: `PascalCase`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jdlar1/siigo-mcp](https://github.com/jdlar1/siigo-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
