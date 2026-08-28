---
trigger: always_on
description: Guidelines for AI agents working in this repository.
---

# AGENTS.md

Guidelines for AI agents working in this repository.

## Overview

This is a pnpm monorepo of **Rome OS apps**. Each app under `apps/` is an independent workspace package that contributes actions, agents, skills, web UIs, APIs, and/or DB schemas to the Rome platform.

- **Language**: TypeScript (ESM)
- **Node**: >=24.11.1 (`.nvmrc`)
- **Package Manager**: pnpm >=10.33.2
- **Testing**: Vitest
- **CI**: GitHub Actions — typecheck + test on push to `main` and all PRs

For the full app development API (app.yaml fields, action patterns, web UI contract, DB/migration model, shadcn/Tailwind setup), read the **app_creation** skill. For lifecycle operations (install, uninstall, enable/disable), read the **app_management** action description.

## Repository Layout

```
rome-apps/
├── apps/                        # All app packages
│   ├── brainstorm/              # Guided product brainstorming agent
│   ├── code-review/             # GitHub PR reviewer (full-stack: DB + API + Web)
│   ├── company-research/        # Company analysis (full-stack)
│   ├── discord-digest/          # Scheduled Discord summaries
│   ├── facebook/                # Facebook scraping
│   ├── learning/                # Learning / vocabulary actions
│   ├── linkedin/                # LinkedIn browser automation
│   ├── memory/                  # Memory & relationship skills
│   ├── news/                    # News ingestion (full-stack)
│   ├── reddit/                  # Reddit radar dashboard (full-stack)
│   ├── rome-table/              # Rome database browser
│   ├── seo/                     # SEO/GEO skill library + actions
│   ├── stock-daily/             # Daily stock-market reports
│   ├── summary/                 # Work summary generator (full-stack)
│   ├── survey/                  # AI-native survey builder
│   ├── utility/                 # Apache-licensed utility skills (6 skills)
│   ├── x-manager/               # X account and content management
│   └── xiaohongshu/             # Xiaohongshu browser automation
├── packages/                    # Shared packages (reserved, currently empty)
├── package.json                 # Root workspace config
├── pnpm-workspace.yaml          # workspaces: apps/*, packages/*
├── .nvmrc                       # Node version pin
└── .github/workflows/
    ├── ci.yml                   # typecheck + test
    ├── rome-publish.yml          # Manual single-app publish (dropdown)
    ├── rome-publish-changed.yml  # Auto-publish apps changed in a commit
    └── rome-publish-stale.yml    # Publish apps whose version isn't live yet
```

---

## Branching Strategy (Git Worktree)

This repository uses **git worktree** to keep the main checkout always on the `main` branch while doing feature work in isolated directories.

### Rules

1. **The main checkout stays on `main`** — never switch branches in the primary repo directory.
2. **Before starting new work**, pull `main` to the latest commit first.
3. **Create a worktree** for every feature / iteration — this gives you a separate directory and branch in one step.
4. **After publish is complete**, remove the worktree and its directory, then update the main checkout to `main` latest.

### Workflow

```bash
# 0. Ensure main is up to date (run in the main repo directory)
cd /path/to/rome-apps
git checkout main
git pull origin main

# 1. Create a worktree + feature branch (from the repo root)
git worktree add ../rome-apps-<feature> -b <feature-branch>
#   e.g. git worktree add ../rome-apps-fix-login -b fix/login-bug

# 2. Work inside the worktree directory
cd ../rome-apps-<feature>
pnpm install          # first time in the worktree
# ... code, build, apply, test, bump version, create PR ...

# 3. After PR is merged & publish is done, clean up
cd /path/to/rome-apps
git worktree remove ../rome-apps-<feature>   # removes worktree + directory
git branch -d <feature-branch>               # delete the local branch
git pull origin main                         # update main to latest
```

> **Tip**: `git worktree list` shows all active worktrees. Always clean up finished worktrees to avoid confusion.

---

## Dev Workflow

> **Goal**: code a change, verify it works locally against the running Rome daemon, then open a PR for review.

### Steps

```
pull main -> create worktree -> code -> build -> install -> test/verify -> bump version -> create PR -> STOP (ask human to review)
```

#### 0. Set Up Worktree

Before writing any code, make sure the main checkout is on the latest `main`, then create a worktree (see [Branching Strategy](#branching-strategy-git-worktree) above). All subsequent steps happen inside the worktree directory.

#### 1. Code

Edit source files under `apps/<appId>/src/` (or `apps/<appId>/actions/` for simple apps without `src/`).

- Follow existing patterns in the same app before inventing new ones.
- Do not introduce cross-app runtime imports. Import only: files inside the same app, Node builtins, declared deps, `@rome-os/app-runtime`, `@rome-os/app-web-sdk`.

#### 2. Build

From the app directory:

```sh
cd apps/<appId>
pnpm install          # if deps changed or first time
pnpm build            # alias for `rome build` — compiles src/ -> dist/
```

`pnpm build` must succeed with no errors. The daemon reads from `dist/`, not `src/`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rome-os/rome-apps](https://github.com/rome-os/rome-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
