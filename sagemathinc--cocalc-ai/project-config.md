---
trigger: always_on
description: Guidance for Claude Code, Gemini CLI, and OpenAI Codex when working in this repository.
---

# AGENTS.md, CLAUDE.md and GEMINI.md

Guidance for Claude Code, Gemini CLI, and OpenAI Codex when working in this repository.

## Repo Facts

- CoCalc-ai source monorepo (TypeScript-heavy) under `src/packages`.
- Workspace management uses pnpm (`src/packages/pnpm-workspace.yaml`).
- Prefer package-local changes and checks over full-repo commands when possible.

## Security Issues

- Before investigating or fixing a suspected vulnerability, read
  [`SECURITY.md`](SECURITY.md).
- Never disclose suspected vulnerability details in a public issue,
  discussion, pull request, commit, or branch before coordinated deployment or
  explicit maintainer approval.
- Use a draft GitHub repository security advisory and its temporary private
  fork for non-public fixes. Do not push the fix to the public repository
  until deployment is coordinated.
- If a security issue is discovered during unrelated work, stop before making
  the details public and alert the user or repository maintainers privately.

## Preferred Commands

- Run these from the repo root unless a command says otherwise.
- Full dev build: `pnpm -C src build:dev`
- Full typecheck: `pnpm -C src tsc`
- Dev/browser environment bootstrap:
  - Lite: `cd src && pnpm dev:lite:env` to print the environment for lite development and browser automation, including the correct `COCALC_API_URL` (typically port `7001`), browser session, and PATH updates. Use `eval "$(pnpm -s dev:lite:env)"` to apply it to the current shell.
  - Hub / full multi-user launchpad: `cd src && pnpm dev:hub:env` to print the corresponding environment for hub-backed development. Use `eval "$(pnpm -s dev:hub:env)"` to apply it to the current shell.
- Prettier (repo-pinned): `pnpm -C src prettier --write <file>`
- Frontend lint (fast): `pnpm -C src lint:frontend`
- Package typecheck (fast): `cd src/packages/<pkg> && pnpm tsc --build`
- Package build: `cd src/packages/<pkg> && pnpm build`
- For `next` / `static`: use `pnpm -C src build:dev` instead of `pnpm -C src build`
- Tests: run focused package/tests first; avoid full test suite unless needed
- Dependency consistency check: `pnpm -C src version-check`

### Fresh Worktrees

- A new git worktree does not have built workspace package outputs. Before running package tests, run `pnpm -C src install`, then build/typecheck the target package (for example, `cd src/packages/<pkg> && pnpm tsc --build`) so TypeScript project references produce their `dist` entrypoints.
- Do not run Jest first in a fresh worktree: imports such as `@cocalc/cloud` can fail misleadingly when the linked workspace package exists but has not been built.
- Use `pnpm -C src build:dev` when preparing a release or when the required dependency scope is unclear; otherwise prefer the package-local build before focused tests.

## Live Hub Env

- Before running live Lite/Launchpad control-plane commands such as `cocalc host ...`, `cocalc project ...`, or browser-driven validation against the local dev hub, refresh the hub env first:
  - `cd src && eval "$(pnpm -s dev:hub:env)"`
- Do this again after restarting the hub, switching between local hub instances, or resuming a stale shell. Otherwise `cocalc` can silently use outdated credentials and fail with misleading auth/control-plane errors.
- When a task depends on upgrading hosts or validating live project-host behavior, assume this step is required unless the current shell definitely just ran it.
- For dangerous CLI operations that require fresh auth in local dev, use `cocalc auth elevate --dev` when the local hub password and database are available. This bootstraps a cookie-backed dev fresh-auth session; raw bearer, API-key, or hub-password auth alone will still fail fresh-auth checks.

## Live Site CLI Auth

- For live staging/prod control-plane work that needs browser-approved fresh auth, use the repo-built one-line bootstrap and wait for the user to approve the printed URLs:
  - Staging: `cd src && node packages/cli/dist/bin/cocalc.js --profile staging --api https://staging.cocalc.ai auth bootstrap --email wstein@gmail.com`
  - Prod: `cd src && node packages/cli/dist/bin/cocalc.js --profile prod --api https://cocalc.ai auth bootstrap --email wstein@gmail.com`
- `auth bootstrap` performs browser login, browser-approved elevation, and `auth status --check` equivalent validation. Elevation lasts 8 hours by default; use `auth elevate --short` only when a short 15-minute window is intentional.
- Do not try to satisfy fresh-auth requirements with API keys, bearer tokens, hub-password auth, or project-scoped credentials. Dangerous operator/admin mutations need a cookie-backed interactive CLI session.

## Live Lite / Browser Env

- Before using `cocalc browser ...` or other live browser automation against the local Lite or Launchpad dev servers, always load the matching env in the current shell:
  - Lite: `cd src && eval "$(pnpm -s dev:lite:env)"`
  - Hub: `cd src && eval "$(pnpm -s dev:hub:env)"`
- This is not optional for reliable targeting. The env sets the correct `COCALC_API_URL`, browser session, project id, auth token, and PATH.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sagemathinc/cocalc-ai](https://github.com/sagemathinc/cocalc-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
