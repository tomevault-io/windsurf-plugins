---
trigger: always_on
description: This is the monorepo that contains all the code and infrastructure for AnalogJS.
---

# AGENTS.md

This is the monorepo that contains all the code and infrastructure for AnalogJS.

## Overview

- **Monorepo** managed by [Nx](https://nx.dev) and [pnpm](https://pnpm.io/)
- Main framework: **AnalogJS** (meta-framework for Angular, powered by Vite)
- Contains multiple apps (Angular, Astro, blog, docs, trpc, etc.) and libraries (shared, card, top-bar, etc.)
- Key packages: `@analogjs/platform`, `@analogjs/vite-plugin-angular`, `@analogjs/vitest-angular`, `@analogjs/vite-plugin-nitro`, `@analogjs/router`, etc.
- Node engines: `^22.0.0 || ^24.0.0`, pnpm `^10.0.0`

## Key Files

- `tsconfig.base.json` - TypeScript path aliases for all packages
- `nx.json` - Nx workspace configuration
- `pnpm-workspace.yaml` - pnpm workspace definition
- `release.config.cjs` - semantic-release configuration
- `.github/workflows/` - CI/CD workflows
- `.githooks/` - git hooks (commit-msg, pre-commit)
- `CONTRIBUTING.md` - full contribution guidelines

## Key Workflows

- **Install dependencies:** `pnpm i`
- **Build all projects:** `pnpm build` (uses Nx)
- **Build** `nx build <project>`
- **Serve main app:** `pnpm dev` or `pnpm start` (runs `nx serve`)
- **Test all projects:** `pnpm test` (runs Vitest via Nx)
- **Format workspace:** `nx format`
- **Lint:** `nx lint <project>`
- **Check formatting:** `nx format:check`
- **Storybook:** `nx storybook <project>`
- **Docs site:** `pnpm nx serve docs-app` (Docusaurus)
- **E2E:** `nx e2e <project>` (Cypress/Playwright)

## Testing a Specific Package

- `nx test <package-name>` (unit tests via Vitest)
- `nx build <package-name>` to verify build
- For E2E: `pnpm e2e`
- Run `nx format:check` to verify formatting

## Project Structure & Conventions

- **Apps:** in `apps/` (e.g., `analog-app`, `astro-app`, `docs-app`, `blog-app`, etc.)
- **Libraries:** in `packages/` (shared code, features, platform, plugins)
- **TypeScript path aliases:** defined in `tsconfig.base.json`
- **Vite config:** each app has its own `vite.config.ts` (see `apps/analog-app/vite.config.ts` for advanced AnalogJS/Vite usage)
- **Release:** Automated with semantic-release through CI, see `release.config.cjs` and `tools/publish.sh`

## Packages → Commit Scopes

| Directory                      | npm Package                     | Commit Scope          |
| ------------------------------ | ------------------------------- | --------------------- |
| `packages/platform`            | `@analogjs/platform`            | `platform`            |
| `packages/router`              | `@analogjs/router`              | `router`              |
| `packages/content`             | `@analogjs/content`             | `content`             |
| `packages/content-plugin`      | `@analogjs/content-plugin`      | `content-plugin`      |
| `packages/vite-plugin-angular` | `@analogjs/vite-plugin-angular` | `vite-plugin-angular` |
| `packages/vite-plugin-nitro`   | `@analogjs/vite-plugin-nitro`   | `vite-plugin-nitro`   |
| `packages/vitest-angular`      | `@analogjs/vitest-angular`      | `vitest-angular`      |
| `packages/nx-plugin`           | `@analogjs/nx-plugin`           | `nx-plugin`           |
| `packages/create-analog`       | `create-analog`                 | `create-analog`       |
| `packages/storybook-angular`   | `@analogjs/storybook-angular`   | `storybook-angular`   |
| `packages/trpc`                | `@analogjs/trpc`                | `trpc`                |
| `packages/astro-angular`       | `@analogjs/astro-angular`       | `astro-angular`       |

## Agent Skills

Reusable agent workflows live in `.agents/skills/`:

- [`fix-issue`](.agents/skills/fix-issue/SKILL.md) - end-to-end flow for resolving a GitHub issue: fetch and understand the issue, create a feature branch off `beta`, implement and verify the fix, then hand off to `open-pr`.
- [`open-pr`](.agents/skills/open-pr/SKILL.md) - commit the current work, push the feature branch, and open a GitHub PR against `beta` filled out from the PR template.

## Contribution Policy

- Use `CONTRIBUTING.md` as the source of truth for base branch, PR requirements, title and commit conventions, supported types/scopes, breaking change notes, and submission expectations.
- Use `.github/PULL_REQUEST_TEMPLATE.md` for PR body structure, including affected scope, test plan, and maintainer-facing merge-strategy recommendations.

## Commit Review Workflow

- Before reviewing branch history, run `git fetch --all`.
- Treat `https://github.com/analogjs/analog.git` as the upstream source of truth and compare the current branch against the relevant `analogjs/*` remote branch.
- If the branch mixes multiple packages or concerns, recommend `git reset --soft <base-commit>` and re-commit the staged changes into smaller, policy-aligned groups.
- Prefer regrouping by affected package or primary package scope using the directory mapping above.
- Before changing GitHub metadata, ask whether the user wants the PR title and description updated. If no PR exists for the branch, ask whether they want one created.
- Treat `Squash merge` as the highly preferred maintainer recommendation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [analogjs/analog](https://github.com/analogjs/analog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
