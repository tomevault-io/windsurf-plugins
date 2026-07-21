---
trigger: always_on
description: These rules are **mandatory** and apply to every task in this repository.
---

# AGENTS.md — Codebase Guide for AI Coding Agents

## Agent Workflow Rules

These rules are **mandatory** and apply to every task in this repository.

### After every change

1. Run `mise run fmt` — auto-fixes formatting (prettier, trailing whitespace, end-of-file). Run first so subsequent checks are clean.
2. Run `yarn build` — the build must pass with no errors before considering the task done.
3. Run `yarn test` — all tests must pass. Fix any failures before finishing.
4. Run `yarn lint` — must pass with zero errors. Fix any lint or formatting issues before finishing.
5. If any command fails, fix the problem and re-run until all four pass cleanly.

### Keeping docs in sync

- **`README.md`** — update whenever you change user-facing behaviour: inputs, outputs,
  supported registries, usage examples, or action semantics.
  - All Markdown tables must use padded, aligned columns: each column padded to the width of
    the widest cell in that column, separator row using `| --- |` style (spaces inside dashes).
- **`AGENTS.md`** — update whenever you change commands, project structure, code style
  conventions, architecture patterns, or testing rules.

---

## Overview

A GitHub Action (TypeScript, Node 24, ESM) that checks whether a Docker image needs
to be rebuilt because its base image has been updated. It compares layer digests across
one or more platforms and supports a wide range of container registries.

```bash
src/index.ts              → thin entrypoint, calls run()
src/main.ts               → action logic, input parsing, output setting, registry routing
src/registry.ts           → abstract ContainerRegistry base class + shared types
src/generic-registry.ts   → GenericRegistry: unified Bearer base (static config or auto-discovery)
src/docker-hub.ts         → DockerHub (extends GenericRegistry)
src/github.ts             → GitHubContainerRegistry (extends GenericRegistry)
src/gitlab.ts             → GitLabContainerRegistry (extends GenericRegistry; registry.gitlab.com)
src/gcr.ts                → GoogleContainerRegistry (extends GenericRegistry; gcr.io + *.gcr.io)
src/quay.ts               → QuayRegistry (extends GenericRegistry)
src/acr.ts                → AzureContainerRegistry (extends GenericRegistry; *.azurecr.io)
src/gar.ts                → GoogleArtifactRegistry (extends GenericRegistry; *.pkg.dev)
src/ecr.ts                → ECRPublicRegistry + ECRPrivateRegistry (extends GenericRegistry)
src/digitalocean.ts       → DigitalOceanContainerRegistry (extends GenericRegistry; registry.digitalocean.com)
src/ocir.ts               → OracleContainerRegistry (extends GenericRegistry; *.ocir.io)
src/auth.ts               → Docker credential resolution
src/image-utils.ts        → image string parsing and layer diff logic
src/token-utils.ts        → HTTP token fetch helpers
```

---

## Commands

### Test

```bash
yarn test                                        # run all tests
yarn test -- --testPathPattern=token-utils       # run a single test file by name
yarn test -- --testPathPattern=image-utils       # another single file
yarn test -- -t "should return diff images"      # run tests matching a name pattern
yarn test -- --testPathPattern=docker-hub -t "getToken"  # file + name filter combined
```

### Lint / Format

```bash
mise run fmt       # hk fix --all — auto-fix formatting (prettier, trailing whitespace, end-of-file)
mise run lint      # hk check --all — full lint suite (markdownlint, actionlint, codespell, etc.)
yarn lint          # prettier --check + eslint (no auto-fix)
yarn format        # prettier --write + eslint --fix (auto-fix)
yarn prettier      # check formatting only
yarn prettier:fix  # fix formatting only
yarn eslint        # lint only (0 warnings allowed)
yarn eslint:fix    # lint + auto-fix
```

### Build

```bash
yarn build           # rollup → dist/index.js  (rimraf dist first)
yarn all             # format + test + build in sequence
```

> **Note:** Tests require `NODE_OPTIONS=--experimental-vm-modules` (already set in the
> `yarn test` script). Do not strip this flag.

---

## TypeScript Configuration

- **Target / Module:** `ES2022` / `NodeNext` — full ESM, no CommonJS
- **Strict mode:** `strict`, `strictNullChecks`, `noImplicitAny`, `noUnusedLocals` all enabled
- `isolatedModules: true` — each file must be independently compilable
- `tsconfig.json` covers `src/` only; test files are excluded (ts-jest handles them separately)
- Node ≥ 24.14.1 required (`.nvmrc` / `.mise.toml`)

---

## Code Style (enforced by Prettier + ESLint)

### Prettier settings (`.prettierrc.json`)

| Setting | Value |
| --- | --- |
| Semicolons | **none** |
| Quotes | **single** |
| Tab width | 2 spaces |
| Trailing commas | **all** (including function params) |
| Print width | 140 |
| Arrow parens | always |
| Bracket spacing | false — `{key: value}` not `{ key: value }` |

### Imports

- Use `import * as core from '@actions/core'` (namespace import) for `@actions/core`
- Use named imports for everything else: `import {Foo, bar} from './foo.js'`
- **Always use `.js` extension** in local import paths, even when the source file is `.ts`:

  ```ts
  import {DockerHub} from './docker-hub.js'   // correct
  import {DockerHub} from './docker-hub'       // wrong — breaks NodeNext resolution

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucacome/docker-image-update-checker](https://github.com/lucacome/docker-image-update-checker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
