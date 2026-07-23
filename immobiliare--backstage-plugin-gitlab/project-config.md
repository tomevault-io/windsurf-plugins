---
trigger: always_on
description: This file provides authoritative guidance for AI agents working on this repository. Read it fully before making any changes.
---

# AGENTS.md — Backstage GitLab Plugin

This file provides authoritative guidance for AI agents working on this repository. Read it fully before making any changes.

---

## Project Overview

This is a **standalone Backstage plugin monorepo** providing GitLab integration for [Backstage](https://backstage.io). It is published to npm as two separate packages:

| Package                   | npm name                                           | Role               |
| ------------------------- | -------------------------------------------------- | ------------------ |
| `packages/gitlab`         | `@immobiliarelabs/backstage-plugin-gitlab`         | Frontend plugin    |
| `packages/gitlab-backend` | `@immobiliarelabs/backstage-plugin-gitlab-backend` | Backend plugin     |
| `packages/dev-sandbox`    | _(private, not published)_                         | Local dev/test app |

**This is NOT a Backstage app** — it is a plugin that gets installed into other people's Backstage apps. Never use `backstage-cli versions:bump`; bump dependencies manually.

- **License**: Apache 2.0
- **Current version**: 6.13.0 (both packages versioned in sync)
- **Backstage compatibility**: Tracks the latest stable Backstage release (`backstage.json`)
- **Package manager**: Yarn 4 (Berry) with `node-modules` linker — use `yarn`, not `npm`

---

## Repository Structure

```
.
├── packages/
│   ├── gitlab/                  # Frontend plugin
│   │   ├── src/
│   │   │   ├── index.ts         # Classic frontend system exports (PRIMARY)
│   │   │   ├── plugin.ts        # Plugin definition + component extensions
│   │   │   ├── routes.ts        # Route reference
│   │   │   ├── translation.ts   # i18n messages
│   │   │   ├── api/             # GitlabCIApi interface + GitlabCIClient implementation
│   │   │   ├── components/      # React components and widgets
│   │   │   │   ├── GitlabCI/    # Main content page (grid layout)
│   │   │   │   ├── gitlabAppData.tsx  # Entity annotation hooks
│   │   │   │   └── widgets/     # Individual cards and tables
│   │   │   └── alpha/           # New frontend system (EXPERIMENTAL)
│   │   │       ├── index.ts     # New system default export
│   │   │       ├── plugin.ts    # createFrontendPlugin definition
│   │   │       ├── extensions.tsx  # EntityContentBlueprint, ApiBlueprint
│   │   │       └── cards.tsx    # EntityCardBlueprint definitions
│   │   ├── config.d.ts          # Frontend config schema
│   │   └── package.json
│   ├── gitlab-backend/          # Backend plugin
│   │   ├── src/
│   │   │   ├── index.ts         # Public exports
│   │   │   ├── plugin.ts        # Backend plugin + catalog module
│   │   │   ├── annotations.ts   # Entity annotation constants
│   │   │   ├── processor/       # GitlabFillerProcessor (catalog)
│   │   │   └── service/         # Express router (API proxy)
│   │   ├── config.d.ts          # Backend config schema
│   │   └── package.json
│   └── dev-sandbox/             # Local dev app (private, not published)
│       └── src/App.tsx          # Uses new frontend system (createApp from @backstage/frontend-defaults)
├── .github/workflows/           # CI/CD pipelines
├── biome.json                   # Biome config (formatting, linting)
├── lefthook.yml                 # Pre-commit hooks and automation
├── tsconfig.json                # Root TypeScript config (covers all packages)
├── backstage.json               # Backstage version compatibility marker
├── commitlint.config.cjs        # Conventional commits enforcement
├── lint-staged.config.cjs       # Pre-commit hook runners
└── .releaserc.json              # Semantic release config
```

---

## Essential Commands

All commands must be run from the **repository root** unless noted otherwise.

### Development

```bash
yarn start          # Start dev-sandbox app (new frontend system)
```

### Build

```bash
yarn build          # tsc (type declarations) + backstage-cli repo build --all
```

The build runs `tsc` first (generates `dist-types/`), then `backstage-cli repo build` for each workspace package.

### Type Checking

```bash
yarn type           # tsc --noEmit (validates all packages via root tsconfig.json)
```

The root `tsconfig.json` includes `packages/*/src` and `packages/*/dev`. Type errors in any package fail this check.

### Testing

```bash
yarn test           # Runs lerna run test:ci across all packages (no watch)
```

Per-package (from the package directory):

```bash
yarn test           # backstage-cli package test (watch mode)
yarn test:ci        # backstage-cli package test --watch false
```

Tests use Jest configured by `@backstage/cli`. There are 5 test suites total:

- `packages/gitlab/src/api/GitlabCIClient.test.ts`
- `packages/gitlab/src/plugin.test.ts`
- `packages/gitlab-backend/src/plugin.test.ts`
- `packages/gitlab-backend/src/processor/processor.test.ts`
- `packages/gitlab-backend/src/service/router.test.ts`

### Linting & Formatting

```bash
yarn style:lint       # biome check packages
yarn style:lint-fix   # biome check --write packages
yarn style:format     # biome format --write packages
```

### Pre-commit Hook

Lefthook runs on every commit. The `lefthook.yml` runs:

1. `yarn biome check --write` on all staged files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [immobiliare/backstage-plugin-gitlab](https://github.com/immobiliare/backstage-plugin-gitlab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
