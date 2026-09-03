---
trigger: always_on
description: Start with `docs/README.md`, the canonical documentation map, then read only the documents that own the behavior being changed. Do not scan the entire reference tree by default.
---

## Reference Routing

Start with `docs/README.md`, the canonical documentation map, then read only the documents that own the behavior being changed. Do not scan the entire reference tree by default.

- Product behavior, commands, runtime requirements, artifacts, or safety: `README.md` and the relevant page under `docs/user/`.
- Contributor workflow and validation: `docs/developer/README.md`, `docs/local-development.md`, and `docs/local-reliability-qa.md`.
- API routes, JSON-RPC, or SSE: `docs/local-ts-api.md`.
- Architecture: the relevant page under `docs/architecture/`, plus `docs/requirements.md` and `docs/decisions.md` when the contract or decision itself changes.
- Plans and status: the relevant active file under `docs/plans/`; use `implemented/` only for historical context.
- TypeScript scripts/dependencies: `package.json`; Python package metadata and tooling: `workers/automation/pyproject.toml`.

Follow links beyond the owning document only when they are needed to resolve a specific contract, dependency, or QA risk.

## How To Run The Project

**Corepack pnpm requirement:** Always invoke pnpm through Corepack as `corepack pnpm ...`. Never run bare `pnpm ...`, even when a global pnpm binary is installed.

**Sandbox false-negative warning:** Localhost requests and process inspection can fail inside the agent sandbox even when JobCtrl services are healthy. A refused or blocked sandboxed `curl`, `ps`, or similar diagnostic is not evidence that a service is down. Before reporting a runtime as unavailable, retry the same read-only probe with the required sandbox escalation and corroborate it with the supervisor status plus an independent listener/process check such as `lsof`. If those sources disagree, report the disagreement and investigate orphaned supervisors or child processes; never collapse contradictory evidence into a “down” diagnosis.

Use `corepack pnpm dev` for the full local development stack. It stops previously tracked JobCtrl process trees for the selected components, then runs the Temporal dev server, TypeScript API, React/Vite web app, and JobCtrl Temporal worker in the foreground so supervised terminals keep the child processes alive. Keep the terminal session open while using the app and stop it with Ctrl-C. Use `corepack pnpm dev:start` only when an explicitly detached background stack is desired in a normal shell.

Known local commands:

- First-run setup: `scripts/install` (guided system checks, including standalone Corepack remediation, plus Node/Python dependencies and Playwright Chromium) or `corepack pnpm dev:setup` (non-interactive dependency sync for already-provisioned machines), then `uv --project workers/automation run jobctrl init` and `uv --project workers/automation run jobctrl doctor`.
- Python CLI: `uv --project workers/automation run jobctrl doctor`, `uv --project workers/automation run jobctrl run`, or targeted `uv --project workers/automation run jobctrl <command>` after dependencies are installed. The full command tree (per-stage runs, `job <url>`, `backup`, `gmail-auth`, `migrate-resume-html`, …) is documented in `README.md` and `docs/user/`. Work-starting commands start Temporal workflows and require the Temporal dev server plus a running JobCtrl worker.
- Full local stack: `corepack pnpm dev` (attached foreground supervisor; preferred for agents and annotation).
- Detached local stack: `corepack pnpm dev:start`, then `corepack pnpm dev:status`, `corepack pnpm dev:logs <name>`, and `corepack pnpm dev:stop`.
- Temporal worker: `uv --project workers/automation run jobctrl worker` (long-lived workflow worker; needs `temporal server start-dev` running).
- TypeScript API: `corepack pnpm api:dev`.
- Web app: `corepack pnpm web:dev`.
- Web preview after build: `corepack pnpm web:preview`.
- Docs site (VitePress over `docs/`): `corepack pnpm docs:dev`, `corepack pnpm docs:build` (fails on dead internal links), `corepack pnpm docs:preview`.

Do not run auto-apply, browser submission, destructive profile/database actions, or commands that submit applications unless the user explicitly asks for that behavior.

## Build, Test, And Lint Commands

Choose the smallest command set from `docs/local-reliability-qa.md` that proves the touched behavior. Reserve `corepack pnpm check` and `corepack pnpm test` for cross-stack, release/high-risk, or explicitly plan-required work. Frontend changes must run their separate web unit/type/E2E/Storybook checks when the touched risk calls for them; the aggregate does not include those suites.

When changing behavior, add or update unit tests for the changed logic. When changing user-facing behavior, local API behavior, browser flows, or UI/UX, include a QA stage that exercises the product path, not only unit tests.

Any major UI/UX regression found by the human must become a QA regression test or an explicitly documented QA checklist item before the work is considered complete.

## Documentation Requirements


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ebarti/JobCtrl](https://github.com/ebarti/JobCtrl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
