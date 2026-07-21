---
trigger: always_on
description: This document is the single source of truth for how to work inside this monorepo. It covers commands, architecture, conventions, testing, and codebase-specific guidance so future AI agents and developers can be effective immediately.
---

# AGENTS.md — GoDaddy JavaScript Monorepo Guide

This document is the single source of truth for how to work inside this monorepo. It covers commands, architecture, conventions, testing, and codebase-specific guidance so future AI agents and developers can be effective immediately.

Repository facts:
- Monorepo managed by pnpm workspaces
- ESM-only (type: "module" across repo)
- Node 24 in CI (.nvmrc), engine >=22 in package.json
- Changesets for versioning and publishing
- GitHub Actions for CI/CD

Quickstart:
- Install: pnpm install
- Build everything: pnpm build
- Test everything: pnpm test
- Lint everything: pnpm lint
- Package-specific: pnpm --filter <pkg> <script>


1) Commands (root and packages)

Root scripts (run from repository root):
- build: pnpm run -r build
- lint: pnpm run -r lint
- test: pnpm run build && pnpm run -r --parallel test
- version: changeset version && pnpm install --prefer-offline
- release: changeset publish
- changeset: changeset
- clean: pnpm run clean:packages && pnpm run clean:root
- clean:packages: pnpm run -r clean
- clean:root: rimraf node_modules

Workspace tips:
- Run a script across all packages: pnpm -r <script> (only runs where script exists)
- Run for a specific package from root: pnpm --filter <name> <script>
  - Example: pnpm --filter @godaddy/react test
- Run a single package in its directory: cd packages/<pkg> && pnpm <script>

Package scripts

eslint-config-godaddy (packages/eslint-config-godaddy)
- lint: eslint .
- pretest: pnpm run --silent lint
- test: echo ok
- clean: rimraf node_modules

eslint-config-godaddy-react (packages/eslint-config-godaddy-react)
- lint: eslint .
- pretest: pnpm run --silent lint
- test: echo ok
- clean: rimraf node_modules

eslint-config-godaddy-typescript (packages/eslint-config-godaddy-typescript)
- lint: eslint .
- pretest: pnpm run --silent lint
- test: echo ok
- clean: rimraf node_modules

eslint-config-godaddy-react-typescript (packages/eslint-config-godaddy-react-typescript)
- lint: eslint .
- pretest: pnpm run --silent lint
- test: echo ok
- clean: rimraf node_modules

biome-config-godaddy (packages/biome-config-godaddy)
- test: echo ok
- clean: rimraf node_modules

@godaddy/app-connect (packages/app-connect)
- dev: tsdown --watch
- build: tsdown
- typecheck: tsc --noEmit
- lint: biome lint --write --unsafe ./src
- check: biome check --write --unsafe ./src
- test: vitest run
- test:watch: vitest
- test:coverage: vitest run --coverage
- release: pnpm build && changeset publish
- prepublishOnly: pnpm build

@godaddy/react (packages/react)
- dev: tsdown --watch
- dev:https: tsdown --watch
- build: tsdown && pnpm dlx @tailwindcss/cli -i ./src/globals.css -o ./dist/index.css --minify
- preview: vite preview
- typecheck: tsc --noEmit
- test: vitest run

@godaddy/localizations (packages/localizations)
- dev: tsdown --watch
- build: tsdown
- typecheck: tsc --noEmit

Common script patterns to run from root:
- Typecheck where available: pnpm -r typecheck
- Watch mode (app-connect, react, localizations): pnpm --filter <pkg> dev
- Format/lint (only where defined): pnpm -r lint or pnpm --filter @godaddy/app-connect check


2) Architecture overview

Monorepo and tooling
- pnpm workspaces: defined in pnpm-workspace.yaml (packages/*)
- ESM throughout: "type": "module" at root and packages
- Node: .nvmrc=24; CI uses Node 24; engines >=22 in root package.json; prefer Node 24 locally to match CI
- CI: .github/workflows
  - CICD on PRs and push to main runs pnpm test
  - Release workflow uses changesets/action to version and publish
  - CodeQL security analysis enabled
- Versioning/publishing: changesets; release PR auto-generated then publish to npm on merge

Packages (top-level purpose)
- eslint-config-godaddy
  - Base ESLint Flat config for JS/ES6 projects
  - Depends on eslint-plugin-jsdoc, eslint-plugin-json, eslint-plugin-mocha, globals
- eslint-config-godaddy-react
  - Extends base; React rules via eslint-plugin-react, jsx-a11y; hooks via FlatCompat for react-hooks
- eslint-config-godaddy-typescript
  - Extends base; TypeScript rules via @typescript-eslint
- eslint-config-godaddy-react-typescript
  - Extends React config; TypeScript rules via @typescript-eslint
- biome-config-godaddy
  - Biome shared config files (biome.json, biome-ts.json) for linting/formatting; users extend these in consumer apps
- @godaddy/app-connect
  - TypeScript verification library for GoDaddy Platform actions and webhook subscriptions
  - Framework adapters for Express and Next.js
  - Uses Vitest, Biome, tsdown build
- @godaddy/react
  - React component library for checkout flows; integrates with commerce APIs
  - Uses tsdown for TS build and Tailwind CLI v4 for CSS build; Vitest for tests; Vite preview
  - Depends on @godaddy/localizations
- @godaddy/localizations
  - Localized strings for checkout components; TS build via tsdown

Key internal dependencies
- @godaddy/react -> @godaddy/localizations (workspace:*)

External tooling per package
- Build: tsdown (TypeScript bundler with DTS output)
- Test: Vitest (react, app-connect)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [godaddy/javascript](https://github.com/godaddy/javascript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
