---
trigger: always_on
description: This document helps AI agents (and humans) understand and work with this repository. Read it before making changes.
---

# Agent Guide

This document helps AI agents (and humans) understand and work with this repository. Read it before making changes.

## Project Overview

`changesets-gitlab` runs the [Changesets](https://changesets.dev) release flow on GitLab CI: it opens/updates a "Version Packages" merge request and, optionally, publishes packages and creates GitLab releases and git tags. It mirrors [`changesets/action`](https://github.com/changesets/action) v2 almost 1:1, adapted to GitLab (merge requests instead of PRs, `GITLAB_TOKEN`/GitLab API instead of Octokit).

- npm package and binary: `changesets-gitlab` (`lib/cli.js`)
- Upstream reference: `changesets/action`, usually checked out next to this repo as `../action`

**Guiding principle: stay as close to `changesets/action` as possible (AMAP). Only diverge for GitLab-specific requirements or genuine bug fixes, and document the divergence.**

## Stack and Tooling

- Node.js `^22.12 || ^24 || >=26`; TypeScript; ESM (`"type": "module"`)
- Yarn 4 (Berry, `node-modules` linker, pinned in `.yarnrc.yml`) — use `yarn`, not `npm`/`pnpm`
- Lint: `@1stg/eslint-config` + `tsc --noEmit`; format: `@1stg/prettier-config`
- Tests: Vitest (Istanbul coverage); type coverage: `type-coverage` at **100%**
- Build: `tsc -p tsconfig.lib.json` → `lib/`
- Hooks: `simple-git-hooks` → `nano-staged` (pre-commit) and `commitlint` (commit-msg)

## Commands

```bash
corepack enable && yarn --immutable # install

yarn lint       # eslint + tsc --noEmit
yarn test       # vitest run (coverage enabled)
yarn build      # build lib/ (tsc)
yarn cli        # run the CLI from source (tsx src/cli)
yarn format     # prettier --write .
yarn typecov    # type-coverage; must stay at 100%
yarn size-limit # bundle-size budget for lib/index.js

# run a single spec
yarn vitest run test/run.spec.ts

# add a changeset for a user-facing change
yarn changeset
```

Always run `yarn lint` and `yarn test` before committing. CI runs `yarn run-s build lint test` on Node 22/24/26.

## Project Structure

| Path                                | Purpose                                                                     |
| ----------------------------------- | --------------------------------------------------------------------------- |
| `src/cli.ts`                        | CLI entry (`commander`); wires all commands                                 |
| `src/main.ts`                       | Default `main` command: the whole release flow                              |
| `src/select-mode.ts`                | `select-mode` command (decides version vs publish)                          |
| `src/version.ts`                    | `version` command (split release)                                           |
| `src/pack.ts`                       | `pack` command (tarballs from a publish plan)                               |
| `src/publish.ts`                    | `publish` command (split release)                                           |
| `src/comment.ts`                    | Shared MR changeset status/comment logic                                    |
| `src/pr-status.ts`                  | `pr-status` command (upstream `/pr-status`)                                 |
| `src/pr-comment.ts`                 | `pr-comment` command (upstream `/pr-comment`)                               |
| `src/run.ts`                        | `runPublish` / `runVersion` (upstream `run.ts`)                             |
| `src/gitlab.ts`                     | `GitLab` client: git + GitLab API (upstream `github.ts`)                    |
| `src/api.ts`                        | Cached Gitbeaker client (`createApi`) and the `GitLabApi` type              |
| `src/env.ts`                        | Environment/input access (`INPUT_*`, `GITLAB_*`, `CI_*`)                    |
| `src/context.ts`                    | GitLab CI context (`projectId`, `ref`, `sha`, …)                            |
| `src/utils.ts`                      | Input/output helpers, exec helpers, `commitChangesSinceBase`, `getUsername` |
| `src/get-changed-packages.ts`       | Changed packages and changed-changeset detection                            |
| `src/read-changeset-state.ts`       | Reads Changesets state (changesets, pre mode)                               |
| `src/constants.ts` / `src/types.ts` | Boolean parsers and shared types                                            |
| `src/index.ts`                      | Public exports                                                              |
| `test/*.spec.ts`                    | Vitest specs (+ `fixtures/`, `__snapshots__/`)                              |

## Upstream File Mapping

`changesets/action` lives in a sibling checkout (usually `../action`). Its root action is `src/index.ts` and each sub-action is `src/<name>/index.ts` (built to `dist/<name>.js`); this port exposes those entry points as CLI commands. Keep the mapping in mind when syncing changes from upstream:

| `changesets/action`                                                     | `changesets-gitlab`                                                                |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [un-ts/changesets-gitlab](https://github.com/un-ts/changesets-gitlab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
