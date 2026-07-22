---
trigger: always_on
description: Keep this file, `AGENTS.md`, up to date when Storybook's architecture, tooling, workflows, or contributor guidance changes.
---

# Storybook Agent Instructions

Keep this file, `AGENTS.md`, up to date when Storybook's architecture, tooling, workflows, or contributor guidance changes.

This file is the canonical instruction source for coding agents. Files like `CLAUDE.md` should point here instead of duplicating instructions.

## Repository Overview

Storybook is a large TypeScript monorepo. The git root is the repo root, the main code lives in `code/`, and build tooling lives in `scripts/`. The default branch is `next`.

- **Base branch**: `next` (all PRs should target `next`, not `main`)
- **Node.js**: `22.22.3` (see `.nvmrc`) — supports `.ts` natively via type stripping (no loader needed)
- **Package Manager**: Yarn Berry
- **Task orchestration**: NX plus the custom `yarn task` runner
- **Linting**: oxlint (root `.oxlintrc.json`, extended by `code/.oxlintrc.json` and `scripts/.oxlintrc.json`; custom rules load via `jsPlugins`). ESLint is no longer used for repo linting — `code/lib/eslint-plugin` remains as the published `eslint-plugin-storybook` package.
- **Formatting**: oxfmt (root `.oxfmtrc.json`)
- **CI environment**: Linux and Windows
- **TS execution**: Migrating from `jiti` to native `node` for running `.ts` files. New scripts should use `node ./path/file.ts` with explicit `.ts` import extensions (enabled by `allowImportingTsExtensions` in tsconfig). Legacy scripts still use `jiti` but should be migrated over time.
- **Type checking**: Per-package checks (`yarn task check`, `scripts/check/check-package.ts`) run on the TypeScript 7 native compiler (the `typescript-native` npm alias); diagnostics are filtered to the checked package. `@storybook/vue3` and `@storybook/svelte` use `vue-tsc` / `svelte-check` (TS 6 based). The workspace `typescript` dependency stays on TS 6 for IDEs and API consumers, so tsconfigs must remain valid for both (e.g. no `baseUrl`).

## Repository Structure

```text
storybook/
├── .github/                      # GitHub configs and workflows
├── .nx/                          # NX workflow state
├── code/                         # Main codebase
│   ├── .storybook/               # Internal Storybook UI config
│   ├── core/                     # Core package published as "storybook"
│   ├── addons/                   # Core addons
│   ├── builders/                 # Builder integrations
│   ├── renderers/                # Renderer integrations
│   ├── frameworks/               # Framework integrations
│   ├── lib/                      # Supporting libraries
│   ├── presets/                  # Webpack-oriented presets
│   └── sandbox/                  # Internal build artifacts
├── scripts/                      # Build and development scripts
├── docs/                         # Documentation
├── test-storybooks/              # Test repos
└── ../storybook-sandboxes/       # Generated sandboxes outside repo
```

## Architecture

### Renderer vs builder vs framework

| Concept   | Role                                  | Example                   |
| --------- | ------------------------------------- | ------------------------- |
| Renderer  | Mounts UI framework to the DOM        | `@storybook/react`        |
| Builder   | Bundles and serves Storybook          | `@storybook/builder-vite` |
| Framework | Renderer + builder + framework config | `@storybook/react-vite`   |

### Core package

The main package is `code/core/src/`. The most important areas are:

- `core-server/` for dev server, static build, and presets
- `manager/` and `manager-api/` for the Storybook UI
- `preview/` and `preview-api/` for story rendering
- `channels/` for manager <-> preview communication
- `csf-tools/` for AST-based story indexing
- `common/` for shared Node.js utilities
- `test/` and `instrumenter/` for testing support

Public exports include:

- `storybook/actions`
- `storybook/preview-api`
- `storybook/manager-api`
- `storybook/theming`
- `storybook/test`

Internal exports include:

- `storybook/internal/core-server`
- `storybook/internal/csf-tools`
- `storybook/internal/common`
- `storybook/internal/channels`

### Key flow

- `.storybook/main.ts` is loaded at startup
- `.storybook/preview.ts` is bundled into preview (TSX for React-based frameworks)
- `.storybook/manager.ts` is bundled into manager
- `*.stories.*` files are indexed by AST before runtime
- Story selection loads the module, prepares the story, and renders it

AST indexing keeps the sidebar fast and prevents one broken story file from breaking the whole UI.

## Common Commands

Run commands from the repository root unless stated otherwise.

For routine agent work, prefer the faster non-production commands first. Add `-c production` only when you need sandbox-related NX tasks or you are explicitly matching CI behavior.

### Install and compile

```bash
yarn
yarn task compile
yarn nx run-many -t compile
yarn nx compile <nx-project-name>
```

### Lint and typecheck

```bash
yarn lint
yarn --cwd code lint:js:cmd <file-relative-to-code-folder> --fix
yarn task check
yarn nx run-many -t check
```

### Development and tests

```bash
cd code && yarn storybook:ui
cd code && yarn storybook:ui:build
yarn test
yarn test:watch
yarn storybook:vitest
```

### Common task scenarios


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [storybookjs/storybook](https://github.com/storybookjs/storybook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
