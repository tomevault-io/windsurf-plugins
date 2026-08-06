---
trigger: always_on
description: This file provides guidance on how to work with the n8n repository.
---

# AGENTS.md

This file provides guidance on how to work with the n8n repository.

## Project Overview

n8n is a workflow automation platform written in TypeScript, using a monorepo
structure managed by pnpm workspaces. It consists of a Node.js backend, Vue.js
frontend, and extensible node-based workflow engine.

## General Guidelines

- Always use pnpm
- When adding comments, keep them concise and to the point - explain the "why"
  in a line or two; don't be overly verbose. Comments should be scoped and
  relevant to the surrounding code, not just to the current task
- We use Linear as a ticket tracking system
- We use Posthog for feature flags
- When starting to work on a new ticket – create a new branch from fresh
  master with the name specified in Linear ticket
- When creating a new branch for a ticket in Linear - use the branch name
  suggested by Linear, **unless it is a security fix** (see Security Fix
  Hygiene below)
- Use mermaid diagrams in MD files when you need to visualise something
- **Developing v3 features:** land normal feature work on `master` behind an
  opt-in flag; introduce breaking changes only on the `3.x` branch. See
  [.github/DEVELOPING_V3.md](.github/DEVELOPING_V3.md).

## Agent Skills and Claude Code Plugin

n8n shared skills live in `.agents/skills/`. Claude Code consumes them through
symlinks in `.claude/plugins/n8n/skills/`; OpenCode reads `.agents/skills/`
directly. Harness-specific overrides remain real directories in the harness
path, such as `.opencode/skills/setup-mcps/`. See
[skills README](.agents/skills/AGENTS.md) for editing and sync guidance.

n8n-specific Claude Code commands and agents live in `.claude/plugins/n8n/` and
are namespaced under `n8n:`. Use `n8n:` prefix when invoking them (e.g.
`/n8n:create-pr`, `/n8n:plan`, `n8n:developer` agent). See
[plugin README](.claude/plugins/n8n/README.md) for structure and details.

## Essential Commands

### Fresh checkout / agent setup

For a fresh checkout (cat-bot, a new hire, any agent verifying the repo
builds), prefer `pnpm agent:setup` over running install + build + tests by
hand. It chains them in one process, caps per-process memory and turbo
concurrency so a 6GB box doesn't OOM, streams all output to
`.agent-setup/<step>.log` (gitignored), and surfaces only a one-line summary
per step plus the tail of the failing log. A machine-readable
`.agent-setup/summary.json` is always written so a backgrounded run is
readable in a single shot — no polling, no scrolling logs.

```bash
pnpm agent:setup                 # install → build → test (full suite)
pnpm agent:setup install         # one step at a time
pnpm agent:setup --json          # JSON summary on stdout (for scripts/agents)
```

### Building
Use `pnpm build` to build all packages. ALWAYS redirect the output of the
build command to a file:

```bash
pnpm build > build.log 2>&1
```

You can inspect the last few lines of the build log file to check for errors:
```bash
tail -n 20 build.log
```

If build outputs or the turbo cache are stale (e.g. after switching branches
or worktrees) but dependencies haven't changed, use `pnpm reset` (lightweight
by default) for a fast recovery: it cleans build outputs and force-rebuilds
(keeping `node_modules` and untracked files). If that doesn't fix your issue,
use `pnpm reset --full`, which also wipes untracked files and reinstalls
dependencies.

### Testing
- `pnpm test` - Run all tests
- `pnpm test:affected` - Runs tests based on what has changed since the last
  commit

Running a particular test file requires going to the directory of that test
and running: `pnpm test <test-file>`.

When changing directories, use `pushd` to navigate into the directory and
`popd` to return to the previous directory. When in doubt, use `pwd` to check
your current directory.

### Code Quality
- `pnpm lint` - Lint code
- `pnpm typecheck` - Run type checks

Always run lint and typecheck before committing code to ensure quality.
Execute these commands from within the specific package directory you're
working on (e.g., `cd packages/cli && pnpm lint`). Run the full repository
check only when preparing the final PR. When your changes affect type
definitions, interfaces in `@n8n/api-types`, or cross-package dependencies,
build the system before running lint and typecheck.

## Architecture Overview

**Monorepo Structure:** pnpm workspaces with Turbo build orchestration

### Package Structure

The monorepo is organized into these key packages:

- **`packages/@n8n/api-types`**: Shared TypeScript interfaces between frontend and backend
- **`packages/workflow`**: Core workflow interfaces and types
- **`packages/core`**: Workflow execution engine
- **`packages/cli`**: Express server, REST API, and CLI commands
- **`packages/editor-ui`**: Vue 3 frontend application
- **`packages/@n8n/i18n`**: Internationalization for UI text
- **`packages/nodes-base`**: Built-in nodes for integrations
- **`packages/@n8n/nodes-langchain`**: AI/LangChain nodes
- **`packages/@n8n/instance-ai`**: "AI Assistant" in the UI, "Instance AI" in code — AI assistant backend. See its `CLAUDE.md` for architecture docs.
- **`@n8n/design-system`**: Vue component library for UI consistency
- **`@n8n/config`**: Centralized configuration management

## Technology Stack


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soulaymanbus/n8nnn](https://github.com/soulaymanbus/n8nnn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
