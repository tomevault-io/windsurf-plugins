---
trigger: always_on
description: Emdash is a local-first, cross-platform Electron app for running multiple AI coding
---

# Project Overview

Emdash is a local-first, cross-platform Electron app for running multiple AI coding
agents in parallel. Each task is isolated in its own Git worktree and can run locally
or over SSH, while the desktop app coordinates provider CLIs, ACP chat sessions,
terminal sessions, issue and PR integrations, diff review, and release packaging.

## Repository Structure

This is a pnpm workspace monorepo. The Electron app lives in
`apps/emdash-desktop/` as `@emdash/emdash-desktop`; shared packages live under
`packages/`. Unless a path is prefixed with `packages/` or another app, app paths
such as `src/...`, `drizzle/`, `scripts/`, and `build/` are relative to
`apps/emdash-desktop/`.

Repo root:

- `.claude/` - Local Claude agent settings for this checkout.
- `.github/` - GitHub issue templates, reusable actions, CI, and release workflows.
- `agents/` - Agent-facing architecture, workflow, convention, integration, and risk docs.
- `apps/emdash-desktop/` - The Electron desktop app.
- `packages/chat-ui/` - Shared transcript and ACP chat renderer with Storybook coverage.
- `packages/core/` - Transport-agnostic runtime primitives, including ACP session logic.
- `packages/plugins/` - Agent provider plugin definitions, hooks, and ACP adapters.
- `packages/shared/` - Shared primitives such as result types, logging, and markdown helpers.
- `packages/ui/` - Shared React UI components, theme tokens, recipes, and primitives.
- `pnpm-workspace.yaml` - Workspace package globs for `apps/*` and `packages/**`.
- Root config files - `package.json`, `nx.json`, `.nvmrc`, `.oxfmtrc.json`,
  `.oxlintrc.json`, and lockfile/configuration owned at the workspace root.

Inside `apps/emdash-desktop/`:

- `build/` - Electron packaging assets; avoid edits unless working on packaging/signing.
- `drizzle/` - Generated Drizzle SQL migrations and metadata.
- `scripts/` - Release, verification, and build support scripts.
- `src/main/` - Electron main process, RPC controllers, services, DB, ACP, PTY, and SSH.
- `src/preload/` - Typed Electron preload bridge exposed to the renderer.
- `src/renderer/` - React app organized around `app/`, `features/`, `lib/`, and tests.
- `src/shared/` - App IPC primitives, provider metadata, events, ACP, MCP, skills, and types.
- `src/types/` - Ambient and cross-cutting TypeScript declarations.
- `tooling/` - App-level development and test infrastructure not bundled into production.

## Build & Development Commands

Use Node `24.14.0` from `.nvmrc` and `pnpm@10.28.2`. Root scripts are powered by Nx
and run package targets in dependency order with local caching where configured.

Install dependencies from the repo root:

```bash
pnpm install
```

Start the full workspace dev setup from the repo root:

```bash
pnpm run dev
```

Start only the Electron app from `apps/emdash-desktop/`:

```bash
cd apps/emdash-desktop
pnpm run dev
pnpm run d
```

Run main-process or renderer-only dev watches from `apps/emdash-desktop/`:

```bash
pnpm run dev:main
pnpm run dev:renderer
```

Run with debug logging from `apps/emdash-desktop/`:

```bash
pnpm run dev:debug
```

Use an isolated development database for schema or migration work:

```bash
EMDASH_DB_FILE=/tmp/emdash-scratch.db pnpm run dev
```

Reset dev databases from `apps/emdash-desktop/`:

```bash
cd apps/emdash-desktop
pnpm run db:reset
```

Build all workspace projects from the repo root:

```bash
pnpm run build
```

Build only the app targets from `apps/emdash-desktop/`:

```bash
cd apps/emdash-desktop
pnpm run build
pnpm run build:main
pnpm run build:renderer
```

Package desktop artifacts locally from `apps/emdash-desktop/`:

```bash
cd apps/emdash-desktop
pnpm run package
pnpm run package:mac
pnpm run package:linux
pnpm run package:win
```

Run formatting, linting, type checks, and tests from the repo root:

```bash
pnpm run format
pnpm run lint
pnpm run typecheck
pnpm run test
```

Run focused database validation from `apps/emdash-desktop/`:

```bash
pnpm run db:setup
pnpm run db:fixtures
pnpm run test:migrations
```

Run Docker-backed SSH development infrastructure from `apps/emdash-desktop/`:

```bash
cd apps/emdash-desktop
pnpm run run:docker-ssh
```

Run Storybook for shared UI packages:

```bash
pnpm --filter @emdash/ui run storybook
pnpm --filter @emdash/chat-ui run storybook
```

Rebuild native Electron dependencies after native dependency changes from
`apps/emdash-desktop/`:

```bash
cd apps/emdash-desktop
pnpm run rebuild
```

Clean and reset app dependencies from `apps/emdash-desktop/`:

```bash
cd apps/emdash-desktop
pnpm run clean
pnpm run reset
```

Deploy releases only when explicitly asked to do release work:

```bash
gh workflow run release-prod.yml --ref main -f arch=both
gh workflow run release-canary.yml --ref main -f arch=both
```

Production releases publish artifacts to GitHub Releases and Cloudflare R2. Canary
releases currently publish to R2 only.

## Code Style & Conventions

- Use `pnpm`; do not introduce npm or yarn lockfile churn.
- Format with `oxfmt`; config is `.oxfmtrc.json`.
- Keep formatted lines near the configured `printWidth` of 100 characters.
- Use 2 spaces, semicolons, single quotes in TS, double quotes in JSX, LF endings,
  trailing commas where valid in ES5, and sorted imports.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [generalaction/emdash](https://github.com/generalaction/emdash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
