---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Repository Structure

This repository is organized as a Yarn v4 monorepo with shared workspaces for plugin packages.

```text
├── examples/
│   ├── 1c/
│   ├── feed-yandex/
│   ├── fulfillment-apiship/
│   ├── payment-robokassa/
│   ├── payment-tkassa/
│   └── payment-yookassa/
├── packages/
│   ├── medusa-1c/
│   ├── medusa-feed-yandex/
│   ├── medusa-fulfillment-apiship/
│   ├── medusa-payment-robokassa/
│   ├── medusa-payment-tkassa/
│   ├── medusa-payment-yookassa/
│   └── utils/
│       └── gorgo-telemetry/
├── scripts/
└── docs/                            # documentation content (built by gorgo/packages/docs)
    ├── medusa-plugins/
    └── tools/
```

> The documentation **site builder** lives in the private `gorgojs/gorgo` repo at `packages/docs`. This repo holds only the docs **content** (MDX); the builder syncs it in at build time.

## Essential Commands

### Monorepo Commands

```bash
yarn install                          # Install all dependencies (triggers husky setup)

 # Update Medusa version across examples
yarn update <version> [example] [-s|--single] [--skip-build]
# e.g.: yarn update 2.14.0 feed-yandex --single --skip-build

# Changesets (versioning & publishing)
yarn changeset                        # Interactively create a changeset
yarn changeset version                # Bump versions and update CHANGELOGs
yarn changeset publish                # Publish packages to npm
```

### Per-Package (Per-Plugin) Commands

Run inside `packages/<name>/`:

```bash
# development
yarn dev                              # Publish locally then start dev watch mode
```

### Per-Example Commands

Run inside `examples/<name>/medusa`:

```bash
yarn install                          # Install all dependencies 
yarn dev                              # Run in development mode
yarn dev:tunnel                       # Run in development mode with tunneling

# testing
yarn test:integration:http            # HTTP integration tests 
yarn test:integration:modules         # Module integration tests
yarn test:unit                        # Unit tests
```

Run inside `examples/<name>/medusa-storefront`:

```bash
yarn install                          # Install all dependencies 
yarn dev                              # Run in development mode
yarn dev:tunnel                       # Run in development mode with tunneling
```

## Commit Conventions

Commits must follow [Conventional Commits](https://www.conventionalcommits.org/) — enforced by commitlint.

**Scope is required** and must be one of:

- Package scope from `packages/` (strip `medusa-` prefix):
  - `1c`
  - `feed-yandex`
  - `fulfillment-apiship`
  - `payment-robokassa`
  - `payment-tkassa`
  - `payment-yookassa`
- Package scope from `packages/utils/` (folder name as-is):
  - `gorgo-telemetry`
- Repo-level:
  - `deps`
  - `release`
  - `docs`
  - `root`

Examples:

```
feat(feed-yandex): add price filter support
fix(payment-robokassa): handle webhook timeout
chore(deps): bump @medusajs/medusa to 2.14.0
```

Scope maps directly to changeset bump type: `feat` → minor, `fix/perf/refactor/docs/revert/test` → patch, breaking (`!`) → major.

## Changelog Generation

`CHANGELOG.md` entries read `<description> by @author in #<pr> (<commit>)` (the `#<pr>` is omitted for direct pushes) and are grouped by **commit type**, not by semver bump (the bump is conveyed by the version number itself). The pipeline:

1. `scripts/generate-changesets.js` turns conventional commits into changesets and stamps each with `commit:` and `author: @<git-author>` (resolved in one GraphQL batch via `scripts/github-authors.js`). The `author:` line makes the credit the **commit author**, not the PR author — `@changesets/changelog-github` otherwise overrides it with the PR author. The description has any trailing ` (#<pr>)` (added by GitHub squash merges) stripped, and the commit body is dropped except a `BREAKING CHANGE:` footer.
2. `changeset version` uses stock `@changesets/changelog-github` (configured in `.changeset/config.json`) to render entries; `scripts/format-changelog.js` (wired into `release:version`) then **reformats** each line to `… by @author in #<pr> (<commit>)` and **regroups** the new version block into sections — Highlights, Breaking Changes, Features, Bug Fixes, …, Chores, Dependencies — deriving the section from each commit's conventional type. It reformats **only the changelogs `changeset version` changed** this release (via `git status`), and within each only the **newest block** — so unreleased packages and any manual edits are left untouched. Idempotent.

> CI pins Node to `24.16.0` in `publish.yml`: Node 24.17+/undici intermittently drops the `@changesets/get-github-info` GraphQL call with `Premature close`, failing `changeset version` (changesets#2115).

Section logic and line reformatting live in `scripts/changelog-utils.js`. A commit lands in **Highlights** if its body contains a `Highlight` trailer (a line that is just `Highlight`). Blocks with no commit link (pre-automation entries) are left untouched. `scripts/rewrite-changelogs.js` is the one-off tool that applied this format to historical changelogs.

## Package Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gorgojs/medusa-integrations](https://github.com/gorgojs/medusa-integrations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
