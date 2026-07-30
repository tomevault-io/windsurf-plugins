---
trigger: always_on
description: | Tool                                          | Purpose                                                                |
---

# Conventions

## Tooling

| Tool                                          | Purpose                                                                |
|-----------------------------------------------|------------------------------------------------------------------------|
| TypeScript 6                                  | Source language (`tsconfig.json` extends `@tada5hi/tsconfig`)          |
| [tsdown](https://tsdown.dev) (rolldown + oxc) | Bundling — emits `dist/index.mjs` + `dist/index.d.mts` + `bin/cli.mjs` |
| Vitest 4 + `unplugin-swc`                     | Test runner; swc transform for decorator metadata                      |
| ESLint 10 + `@tada5hi/eslint-config` v2       | Linting (flat config in `eslint.config.mjs`)                           |
| `@tada5hi/commitlint-config`                  | Conventional Commits enforcement                                       |
| Husky 9                                       | `commit-msg` hook runs commitlint                                      |
| release-please + [`tada5hi/monoship`](https://github.com/tada5hi/monoship) | Automated release PRs + npm publish on merge |
| VitePress 1                                   | Docs site under `docs/`                                                |

## Validation & Error Handling

- **Validation**: no schema-validation library is used. Public API functions take loose input objects and trust internal callers.
- **Errors**: typed via `TypeormExtensionError` → `DriverError` / `OptionsError` (`src/errors/`). Throw a typed error only when a consumer might reasonably want to catch it (e.g. unsupported driver, missing data source). Otherwise let TypeORM / native-driver errors propagate.
- **Validation location**: context-builder functions (`buildDatabaseCreateContext`, `buildDataSourceOptions`, `resolveSeederConfig`) are the choke points where defaults are applied and missing values raise `OptionsError`.

## Workflow

- After changing source, run `npm run build` (catches TS errors that ESLint misses) and `npm test`.
- After changing source or tests, run `npm run lint` on the affected files (or `npm run lint:fix`).
- When changing user-facing behavior (CLI flags, public API signatures, env var names, seeder/factory contract), update both `README.MD` and the matching page in `docs/guide/`.
- Adding a new TypeORM driver: add a dialect folder `src/database/core/<name>/` (`statements.ts` + `module.ts`), an adapter in `src/database/adapters/<name>.ts`, and a row in `src/database/registry.ts`. Then add a spec under `test/unit/database/core/` (using the memory connection factory from `test/data/database/`) and a docs entry.

## Code Style

- **Module format**: ESM-only (`"type": "module"` + `module: ESNext`). tsdown emits a single `.mjs` bundle. A small tsdown plugin (`typeormDeepImportExtension` in `tsdown.config.ts`) rewrites bare `typeorm/<deep>` imports to add `.js` for Node's strict ESM resolver.
- **Indentation**: 4 spaces, LF line endings, UTF-8, final newline, trim trailing whitespace (`.editorconfig`).
- **Linting**: `@tada5hi/eslint-config` v2 (flat config, ESLint 10). Project-local overrides in `eslint.config.mjs`:
  - `class-methods-use-this: off`
  - `no-shadow`, `no-use-before-define`, `@typescript-eslint/no-unused-vars`, `@typescript-eslint/no-use-before-define`: off.
  - Ignores `dist/**`, `bin/**`, declaration files, vitepress build output.

## Naming Conventions

- **Files**: kebab-case (`data-source.ts`, `file-path.ts`).
- **Folders**: kebab-case (`data-source/`, `cli/commands/database/`).
- **Classes**: PascalCase (`SeederExecutor`, `SeederFactoryManager`).
- **Interfaces**: an interface that is implemented by a class is prefixed with `I` (`IDatabaseDialect` → `class PostgresDialect implements IDatabaseDialect`). Pure data shapes — options, contexts, records like `Environment`, `TSConfig` — carry no prefix. Existing public contracts that predate this rule (e.g. `Seeder`, implemented by consumer seed classes) are grandfathered: renaming them would break public API; apply the prefix to new interfaces only.
- **Functions**: camelCase verb-first (`createDatabase`, `findDataSource`, `useDataSource`, `applyQuery`, `buildDatabaseCreateContext`).
- **Hook-style accessors**: `use*` (`useDataSource`, `useEnv`, `useSeederFactoryManager`). These return cached/singleton state.
- **Registry mutators**: `set*` / `has*` / `unset*` / `reset*` (see `src/data-source/singleton.ts`, `src/env/module.ts`).
- **Context/options types**: `XContextInput` (loose, user-supplied) → `XContext` (resolved) → driver receives the resolved one. Example: `DatabaseCreateContextInput` → `DatabaseCreateContext`.
- **CLI commands**: `defineCLI<Noun><Verb>Command()` factory returning a citty `defineCommand` instance (e.g. `defineCLIDatabaseCreateCommand`). Parent commands that only group subcommands use the same `defineCLI<Noun>Command()` shape (e.g. `defineCLIDatabaseCommand`).

## File Organization

- Each domain folder under `src/` has an `index.ts` **barrel** that re-exports everything public from sibling files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tada5hi/typeorm-extension](https://github.com/tada5hi/typeorm-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
