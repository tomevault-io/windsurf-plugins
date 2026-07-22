---
trigger: always_on
description: Trust these instructions. Only search the codebase if information here is incomplete or found to be in error (notify the user if an error is found). Following these steps will minimize failed commands and CI rejections.
---

# Beachball copilot instructions

Trust these instructions. Only search the codebase if information here is incomplete or found to be in error (notify the user if an error is found). Following these steps will minimize failed commands and CI rejections.

## What this repo is

Beachball is a CLI tool for automating semantic version bumping, changelog generation, and npm publishing in monorepos and single-package repos. This repo is a **Yarn 4 monorepo** of ~2,000 files. The main deliverable is the `beachball` npm package.

- **Runtime:** Node.js `>=22.18.0` (repo developed on Node 22; CI also runs Node 24). TypeScript files can be run directly with `node` (no extra flags).
- **Language:** TypeScript `~6.0`. The `beachball` and `p-graph` package output is CommonJS (other packages use ESM).
- **Package manager:** Yarn 4 (Berry), `nodeLinker: node-modules`. Some dependency versions are shared via Yarn catalogs (`catalogs` in [.yarnrc.yml](./.yarnrc.yml)). Never use `npm install`.
- **Task runner:** [lage](https://microsoft.github.io/lage/) orchestrates `build`, `test`, `lint` across workspaces (config: [lage.config.js](./lage.config.js)).
- **Testing:** Jest 30 via Babel.
- **Linting/formatting:** ESLint 9 flat config + Prettier.
- **Dependency consistency:** syncpack + depcheck.

## Branches

- `main` (default): development for beachball v3 (alpha). Breaking changes allowed.
- `v2`: current stable release. Non-breaking changes should target `v2`.

## Build and validation commands

**Always run `yarn --immutable` first** if dependencies may be out of date.

## Repo root commands

When validating the entire repo, run the following commands in order:

| Step             | Command                 | Notes / timing (validated)                           |
| ---------------- | ----------------------- | ---------------------------------------------------- |
| Install          | `yarn --immutable`      | Required before any build/test/lint.                 |
| Build            | `yarn build`            | ~8s. Runs `lage build bundle` across all workspaces. |
| Format (fix)     | `yarn format`           | Auto-fixes formatting.                               |
| Lint             | `yarn lint`             | ~10s. Runs eslint + depcheck + syncpack via lage.    |
| Test (all)       | `yarn test`             | Runs all workspace tests via lage.                   |
| Update snapshots | `yarn update-snapshots` | Use after intentional output changes.                |

**Do NOT run `jest` or `tsc` directly from the repo root.** `yarn build` and `yarn lint` must succeed with zero warnings.

### Package-level commands

When developing in a single package, **use these commands first** before building/validating the entire repo. Run `cd packages/<name>` then:

| Task                                            | Command                    |
| ----------------------------------------------- | -------------------------- |
| Build a single package                          | `yarn build`               |
| Test (packages other than `beachball`)          | `yarn test`                |
| (`beachball`) All tests, correct order          | `yarn test:all`            |
| (`beachball`) All unit tests                    | `yarn test:unit` (~3s)     |
| (`beachball`) All functional tests              | `yarn test:func`           |
| (`beachball`) All E2E tests                     | `yarn test:e2e` (slow)     |
| Single test file/name (wraps jest), ANY package | `yarn test <path or name>` |
| Update snapshots                                | `yarn update-snapshots`    |

In the `beachball` package, do NOT use `test:*` commands to run a single test. Use `yarn test <path or name>`.

Prefer scripts over running binaries directly. If you must run a binary such as `jest`, use `yarn run -T <binary>`.

### Required before each commit

Run all of: `yarn build`, `yarn test`, `yarn lint`, `yarn format`.

### Required before creating a PR

- Use the `/beachball-change-file` skill to generate a Beachball change file if needed. Sometimes it's not needed, but verify by running `yarn checkchange` (CI also runs this).
- Check whether documentation site or CLI help text updates are needed.

## Repo layout

- `packages/beachball` — main package (see architecture below).
- `packages/p-graph` — promise graph runner, used by `beachball`.
- `packages/proper-changelog` — changelog/release notes helper.
- `packages/esrp-npm-release` — helper for the Microsoft release process.
- `scripts` — repo-internal scripts (`@microsoft/beachball-scripts`); shared build/test/eslint config lives in [scripts/config](./scripts/config).
- `actions/*` — GitHub Action definitions (`check-for-modified-files`, `install-beachball`, `should-release`); their `dist/` is committed and must be rebuilt via `yarn build`+`yarn bundle` if `src` changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/beachball](https://github.com/microsoft/beachball) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
