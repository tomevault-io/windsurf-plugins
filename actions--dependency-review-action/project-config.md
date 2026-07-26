---
trigger: always_on
description: Trust these instructions. Only search the codebase if information here is incomplete or found to be in error.
---

# Copilot Coding Agent Instructions

Trust these instructions. Only search the codebase if information here is incomplete or found to be in error.

## Repository Overview

**dependency-review-action** is a GitHub Action (TypeScript/Node.js 24) that scans pull requests for dependency changes, raising errors for vulnerabilities or invalid licenses. It queries the GitHub Dependency Review API, evaluates changes against configured rules, and produces job summaries and PR comments. The action entry point is `dist/index.js` (bundled via `ncc`). The repo is small (~15 source files, ~15 test files).

## Build & Validation Commands

For CI-parity installs and local validation, run `npm ci --ignore-scripts` before other commands. This is the install step used in CI; release workflows may follow different install instructions (see CONTRIBUTING).

| Task         | Command                   | Notes                                                                                                                                    |
| ------------ | ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Install      | `npm ci --ignore-scripts` | ~45s. Use `--ignore-scripts` for CI-parity installs; release workflows may use `npm i` per CONTRIBUTING.                                 |
| Build        | `npm run build`           | Compiles `src/*.ts` → `lib/*.js` via `tsc -p tsconfig.build.json`. ~5s.                                                                  |
| Test         | `npm test`                | Runs Jest. ~8s. All tests should pass.                                                                                                   |
| Lint         | `npm run lint`            | ESLint on `src/**/*.ts`. Ignore the TS version warning—it still passes.                                                                  |
| Format check | `npm run format-check`    | Prettier check on `**/*.ts`.                                                                                                             |
| Format fix   | `npm run format`          | Auto-fix formatting with Prettier.                                                                                                       |
| Package      | `npm run package`         | Bundles the action entrypoint (`package.json#main`) → `dist/index.js` via `ncc`. ~7s. Do NOT include `dist/` changes in non-release PRs. |
| All          | `npm run all`             | Runs: build → format → lint → package → test (in that order).                                                                            |

### Validation Sequence After Making Changes

Always run these commands in this order to validate changes:

```sh
npm run build
npm run format-check
npm run lint
npm test
```

If format-check fails, run `npm run format` to auto-fix, then re-check.

### CI Checks (`.github/workflows/ci.yml`)

CI runs on PRs (excluding `**.md` changes) with Node 24:

1. **test** job: `npm ci --ignore-scripts` → `npm test`
2. **lint** job: `npm ci --ignore-scripts` → `npm run format-check` → `npm run lint`

Additional workflows: `dependency-review.yml` (self-test), `codeql.yml` (CodeQL analysis), `stale.yaml` (stale issues).

## Project Layout

```
src/                  # TypeScript source (edit these files)
  main.ts             # Entry point — orchestrates the action (532 lines)
  schemas.ts          # Zod schemas & TypeScript types for all data structures
  config.ts           # Reads action inputs + external YAML config
  dependency-graph.ts # GitHub API client for dependency diff
  filter.ts           # Filters changes by severity, scope, allowed advisories
  licenses.ts         # License validation against allow/deny lists
  deny.ts             # Package/group deny-listing logic
  purl.ts             # Package URL (PURL) parser
  spdx.ts             # SPDX license expression handling
  scorecard.ts        # OpenSSF Scorecard integration
  summary.ts          # Summary/report generation (736 lines, largest module)
  comment-pr.ts       # Posts/updates PR comments with results
  git-refs.ts         # Resolves base/head git refs from event payload
  utils.ts            # Shared utilities (Octokit client, grouping helpers)
lib/                  # Compiled JS output (from `npm run build`). Gitignored.
dist/                 # Bundled action (from `npm run package`). Committed but do NOT include changes in normal PRs - only pull requests which are creating new releases should have these files changed.
__tests__/            # Jest test files (*.test.ts)
  test-helpers.ts     # setInput()/clearInputs() helpers for test env vars
  fixtures/           # YAML config samples and factory helpers
    create-test-change.ts       # Factory for mock Change objects
    create-test-vulnerability.ts # Factory for mock vulnerability objects
scripts/              # Dev/debug utilities (scan_pr for manual testing, create_summary.ts for preview)
action.yml            # Action metadata — inputs, outputs, and `runs.main: dist/index.js`
```

### Configuration Files


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [actions/dependency-review-action](https://github.com/actions/dependency-review-action) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
