---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> Auto-loaded by Claude Code at session start. Read this first.

## Project Overview

LFX One is a Turborepo monorepo containing an Angular 20 SSR application with stable zoneless change detection and Express.js server.

## Working mode

You have full file-edit authority in this session — different from a Cowork session where you generate prompts for someone else to execute. For pre-edit hygiene checks (re-read files, type-check after multi-file changes, etc.) invoke the `/develop` skill.

**Lean on subagents.** Use the `Agent` tool for broad searches (`Explore`), independent parallel investigations (multiple Agent calls in one message), and context-heavy reads that would bloat the main thread. For the LFX post-commit audit, launch the `lfx-self-serve-code-reviewer` and `lfx-self-serve-learnings-reviewer` subagents in parallel (`subagent_type: lfx-self-serve-code-reviewer` / `subagent_type: lfx-self-serve-learnings-reviewer`, both with `run_in_background: true`) — their definitions in `.claude/agents/` carry the full review playbook. Default to delegating when the task is wide, parallel, or read-heavy.

## Domain language

Use these naturally — do not paraphrase:

- **PCC** — Project Control Center
- **ED** — Executive Director
- **Admin Mode** — privileged view variant for EDs and admins
- **Affiliation** — contributor's company/org link
- **L2** — second-level navigation pattern
- **Personas** — Contributor, Maintainer, ED, Board Member

When a feature affects multiple personas differently, flag it explicitly.

## Quick Start

**Prerequisites:** Node.js ≥22, Yarn 4.x (via corepack), Docker (for the local microservice stack).

For first-time setup (1Password env vars, microservice stack, etc.) invoke the `/setup` skill — it handles prerequisites, clone, install, env vars, and the dev server.

## Commands

All commands run from the repo root via Turborepo:

| Command             | Purpose                                             |
| ------------------- | --------------------------------------------------- |
| `yarn start`        | Angular dev server with hot reload (via Turbo)      |
| `yarn build`        | Production build (all packages)                     |
| `yarn lint`         | Lint + auto-fix across the monorepo                 |
| `yarn lint:check`   | Lint without auto-fix (CI mode)                     |
| `yarn check-types`  | TypeScript type-check only (no emit)                |
| `yarn format`       | Prettier write across the repo                      |
| `yarn format:check` | Prettier check (CI mode)                            |
| `yarn e2e`          | Playwright E2E suite (headless)                     |
| `yarn e2e:ui`       | Playwright in interactive UI mode                   |
| `yarn e2e:headed`   | Playwright headed, visible browser                  |
| `yarn commitlint`   | Validate commit message against Angular conventions |

> For manual commands, prefer `yarn` over `npx` — the repo pins Yarn 4.x through `packageManager`, so `npx` can resolve to the wrong binary. Repo-managed tooling (e.g. `.husky/pre-commit` invokes `npx lint-staged`) may still use `npx` where already configured.

### Reset / cleanup

```bash
yarn ng cache clean        # Angular CLI cache (uses the workspace-local ng)
yarn turbo clean           # Turborepo build cache (turbo is a local devDep)
rm -rf node_modules && yarn install   # nuclear
```

Hot reload silent? Likely `inotify` watcher limit — `sudo sysctl fs.inotify.max_user_watches=524288`.

## Monorepo Structure

```text
lfx-self-serve/
├── apps/
│   └── lfx-one/              # Angular 20 SSR application with stable zoneless change detection
│       ├── src/app/
│       │   ├── layouts/      # Layout components (main-layout, profile-layout)
│       │   ├── modules/      # Feature modules (see Feature Modules section)
│       │   └── shared/       # Shared application code
│       │       ├── components/   # UI components (PrimeNG wrappers + LFX primitives)
│       │       ├── directives/   # Custom directives (on-render, scroll-shadow)
│       │       ├── guards/       # Route guards (auth, writer, executive-director)
│       │       ├── interceptors/ # HTTP interceptors (authentication)
│       │       ├── pipes/        # Custom pipes
│       │       ├── providers/    # App providers (datadog-rum, feature-flag, runtime-config)
│       │       ├── services/     # Frontend services
│       │       ├── strategies/   # Routing strategies (custom-preloading)
│       │       └── utils/        # App utilities (console-override, download-card, http-error, etc.)
│       ├── src/server/       # Express.js SSR server
│       │   ├── constants/    # Server-only constants
│       │   ├── controllers/  # Route controllers
│       │   ├── errors/       # Custom error classes (base, authentication, microservice, service-validation)
│       │   ├── helpers/      # Server helpers (api-gateway, error-serializer, http-status, ics, meeting, poll-endpoint, query-service, url-validation, validation)
│       │   ├── middleware/   # Express middleware (auth, error-handler, rate-limit)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linuxfoundation/lfx-self-serve](https://github.com/linuxfoundation/lfx-self-serve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
