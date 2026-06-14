---
trigger: always_on
description: `ghg` is a TypeScript CLI for GitHub workflow automation. It extends day-to-day GitHub work with notification triage, pull request helpers, profile/config management, label syncing, repository governance, and repository insights. The runtime is Node.js, the CLI layer is Commander, and the codebase uses a layered structure:
---

# AGENTS.md

## 1. Overview

`ghg` is a TypeScript CLI for GitHub workflow automation. It extends day-to-day GitHub work with notification triage, pull request helpers, profile/config management, label syncing, repository governance, and repository insights. The runtime is Node.js, the CLI layer is Commander, and the codebase uses a layered structure:

CLI entrypoint -> command modules -> services -> API/core helpers -> shared types

The project now has a human-first terminal UX with explicit `--json` support. Status messaging goes through the shared logger and renderer stack in `src/core/`.

## 2. Repository Structure

```text
src/
  api/
    client.ts         # shared GitHub HTTP client, auth, pagination, error mapping
    commits.ts
    contents.ts
    insights.ts
    issues.ts
    labels.ts
    notifications.ts
    pr.ts
    pulls.ts
    repos.ts
    rulesets.ts
  cli/
    ascii.ts          # banner used in help output
    index.ts          # root Commander program, global flags, error boundary
  commands/
    activity.ts
    config.ts
    gh.ts
    insights.ts
    labels.ts
    mentions.ts
    notifications.ts
    ping.ts
    pr.ts
    profile.ts
    repos.ts
  core/
    command.ts        # shared command runner
    config.ts         # env + credentials + profile resolution
    constants.ts
    dates.ts
    errors.ts
    git.ts
    io.ts
    logger.ts
    output-state.ts
    output.ts
    progress.ts
    prompt.ts
    spinner.ts
    theme.ts
  services/
    repos/
      govern.ts
      index.ts
      inspect.ts
      label.ts
      report.ts
      retire.ts
    config.ts
    insights.ts
    labels.ts
    notifications.ts
    pr.ts
    profile.ts
    stack.ts
  types/
    index.ts
    notifications.ts
  tui/
    app.ts            # Full-screen TUI runtime
    index.ts          # TUI entry and renderer wiring
    types.ts          # TUI-specific types
    operations/       # Workspace operation definitions
      index.ts        # Concatenates all workspace arrays
      shared.ts       # Input helpers (text, numberValue, targetOptions, etc.)
      dashboard.ts
      notifications.ts
      labels.ts
      prs.ts
      review.ts
      milestones.ts
      projects.ts
      issues.ts
      repositories.ts
      insights.ts
      workflow.ts
      cache.ts
      run.ts
      profile.ts
      config.ts
      utility.ts
      release.ts
    layout.ts         # Screen layout calculations
    mouse.ts          # Mouse event parsing
    render.ts         # Ink-based rendering
    state.ts          # Dashboard and context state
    status.ts         # Status bar items
  env.d.ts
templates/
  base.json
  conventional.json
  github.json
tests/
  unit/
    api/
    cli/
    commands/
    core/
    services/
scripts/
  clean.sh
package.json
tsconfig.json
tests/tsconfig.json
vite.config.ts
eslint.config.mjs
.prettierrc.json
VERSION
```

- Add new commands in `src/commands/`. Each module exports a `register(program)` entry.
- Put business logic in `src/services/`. Services orchestrate API calls, git helpers, filesystem access, and rendering decisions.
- Put GitHub REST wrappers in `src/api/`. Never call `fetch` outside `src/api/client.ts`.
- Put shared terminal UX primitives in `src/core/`. Human output is centralized there.
- Keep shared interfaces in `src/types/`.

## 3. Build, Test, and Local Workflows

```bash
pnpm install
pnpm build
pnpm start
pnpm test
pnpm test -- --run
pnpm test:coverage
pnpm lint
pnpm format
pnpm format:check
pnpm typecheck
npx tsc --noEmit -p tests/tsconfig.json
pnpm clean
bash scripts/clean.sh
```

`pnpm build` produces `dist/index.js` and copies `templates/` into `dist/`.

## 4. Architecture and Boundaries

- `src/cli/index.ts` owns global flags, help behavior, command registration, and the top-level error boundary.
- Command modules should stay thin. Parse flags, prompt when needed, then hand off to a service.
- Services contain the main workflow logic. They may render user-facing output through `core/output`, `core/spinner`, `core/progress`, and `core/logger`.
- API modules wrap GitHub endpoints and use the shared client for headers, auth, request methods, pagination, and HTTP-to-error mapping.
- Config resolution is centralized in `src/core/config.ts`. Do not import `dotenv/config` anywhere else.
- Shared constants belong in `src/core/constants.ts`.
- Shared error types belong in `src/core/errors.ts`.

## 5. Commands and Product Surface

Current command families:

- `ghg notifications ...`
- `ghg activity`
- `ghg mentions`
- `ghg labels ...`
- `ghg repos ...`
- `ghg insights ...`
- `ghg pr ...`
- `ghg profile ...`
- `ghg config ...`
- `ghg gh ...`
- `ghg ping`

Repository governance lives under `ghg repos`:

- `inspect`
- `govern`
- `label`
- `retire`
- `report`

The root CLI supports `--json` and `--theme <dark|light|auto>`.

## 6. Code Style

TypeScript formatting is strict and Prettier-driven:

- 2-space indentation
- double quotes
- semicolons required
- trailing commas in multi-line literals/imports

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airscripts/ghitgud](https://github.com/airscripts/ghitgud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
