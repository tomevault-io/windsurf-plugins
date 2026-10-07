---
trigger: always_on
description: Guidance for Claude Code (and any AI coding agent) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (and any AI coding agent) working in this repository.

This is an **open source project**. Most people opening it are outside contributors. Keep changes small, focused and easy to review.

## What this project is

`eslint-config-airbnb-extended` is a maintained successor to the unmaintained Airbnb ESLint configs (`eslint-config-airbnb`, `eslint-config-airbnb-base`, `eslint-config-airbnb-typescript`). It supports:

- ESLint 9+ **flat config only**. Legacy `.eslintrc*` is not supported.
- TypeScript, React, Next.js and Node.
- All plugins bundled ("batteries included"). Users install only `eslint` and this package.

Docs: https://eslint-airbnb-extended.nishargshah.dev (source in `docs/`).

## Repo layout

pnpm monorepo (`pnpm-workspace.yaml`):

| Path                                     | Package                               | Purpose                                                                        |
| ---------------------------------------- | ------------------------------------- | ------------------------------------------------------------------------------ |
| `packages/eslint-config-airbnb-extended` | `eslint-config-airbnb-extended` (npm) | The ESLint config                                                              |
| `packages/create-airbnb-x-config`        | `create-airbnb-x-config` (npm)        | CLI that writes an `eslint.config.mjs` into a user's project                   |
| `apps/build-templates`                   | private                               | Generates the `eslint.config.mjs` templates the CLI downloads                  |
| `docs`                                   | private                               | VitePress docs site                                                            |
| `configs/*`                              | private                               | Shared eslint / prettier / lint-staged / tsconfig / vitest for every workspace |

## Branches and PRs

- **Always open PRs against `canary`.** Never target `master`.
- `master` holds released code. The CLI downloads templates from `master`, so template changes reach users only after a release.
- Fill in `.github/pull_request_template.md`.
- Use Conventional Commits: `feat:`, `fix:`, `docs:`, `chore:`, ...

## Setup and commands

Use **pnpm only**. npm and yarn are blocked in `engines`. The Node version is in `.nvmrc`. The pnpm version is in `packageManager` in the root `package.json`.

```bash
pnpm install
pnpm build                  # build all workspaces (lint needs this: shared lint config imports the built package)
pnpm config:build           # build only the ESLint config package
pnpm cli:dev                # run the CLI from source
pnpm templates:build        # regenerate apps/build-templates/templates
pnpm docs:dev               # docs dev server
pnpm lint / pnpm lint:fix
pnpm format / pnpm format:fix
pnpm typecheck
pnpm test                    # run all tests (Vitest)
pnpm test:ui                 # Vitest UI
pnpm --filter <pkg> test:coverage   # coverage report for one package
pnpm script:lint --for=check                                      # prettier + eslint + tsc (pre-push hook)
pnpm script:lint --for=ci                                         # what CI runs, after `pnpm build`
pnpm script:lint --filter=create-airbnb-x-config --no-typecheck   # one workspace only
pnpm lint:inspector                                               # open @eslint/config-inspector
```

Both npm packages have a Vitest suite in their `tests/` folder (same folder layout as the source, shared config in `configs/vitest-config`). Rule, config and plugin snapshots live in `tests/**/__snapshots__`. If you change a rule or config on purpose, run `pnpm test -u` and commit the updated snapshots. A change is valid when build, test, lint, format and typecheck all pass. CI (`.github/workflows/validate-pr.yml`) runs `pnpm build`, `pnpm script:lint --for=ci` and `pnpm test`.

Git hooks: pre-commit runs `lint-staged`, pre-push runs `script:lint --for=check`.

## ESLint config package

Path: `packages/eslint-config-airbnb-extended`. Built with tsdown from two entries:

- `index.ts` → `eslint-config-airbnb-extended` (Extended config)
- `legacy.ts` → `eslint-config-airbnb-extended/legacy` (Legacy config)

Imports use the `@/` alias for the package root.

### Public API (`index.ts`)

| Export       | Shape                                                                                                                                                                                   |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `configs`    | `configs.{base,react,next}.{recommended,typescript,all}`, `configs.node.recommended`                                                                                                    |
| `rules`      | `rules.base.*`, `rules.node.*`, `rules.react.*`, `rules.next.*`, `rules.typescript.*` (includes strict sets: `base.importsStrict`, `react.strict`, `typescript.typescriptEslintStrict`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eslint-config/airbnb-extended](https://github.com/eslint-config/airbnb-extended) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
