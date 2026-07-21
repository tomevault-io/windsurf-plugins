---
trigger: always_on
description: Guidance for AI coding agents (Claude Code, Cursor, Copilot, etc.) working in this repository.
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, Cursor, Copilot, etc.) working in this repository.

## What this is

Hammerkit is a containerized build tool (CLI distributed as `dist/index.js`). It reads a `.hammerkit.yaml` build file, plans a graph of tasks and services, and executes them locally, in Docker, or on Kubernetes. Source/output files are explicitly declared per task, which is what enables hammerkit's incremental caching and "store/restore" workflow for CI.

User-facing docs live at https://no0dles.gitbook.io/hammerkit/ (see [SUMMARY.md](SUMMARY.md) for the structure).

## Working agreement

Read [VALUES.md](./VALUES.md) first. That file covers what hammerkit optimizes for; this file covers agent-specific operating rules.

How to work in this repo (see the `do-the-work` skill for detail):

- **Do the work; don't avoid it.** When the task is to fix something — CI, a test,
  the build, a check — fixing the root cause *is* the deliverable. Never resolve a
  failure by skipping, disabling, excluding, `.skip`/`xfail`, weakening an
  assertion, or stopping early (unless the user explicitly asks).
- **Ask implementation questions, not permission to avoid work.** Design and
  ambiguity questions are welcome; "can I skip/disable/leave this?" is not — that's
  the work, do it. Escalate only when genuinely blocked (access, a product
  decision, contradictory requirements), and propose the fix path.
- **Don't make the user repeat himself.** Read [.agents/FEEDBACK.md](.agents/FEEDBACK.md)
  before working and apply it; when corrected, append the correction there so it's
  not re-litigated.

## Commands

- `npm run build` — `tsc -b tsconfig.json`, outputs to `dist/`
- `npm test` — runs jest across `src/**/*.spec.ts`. Tests run with `maxConcurrency: 1` and a 45s timeout; expect a full run to be slow.
- `npm run lint` / `npm run format` — eslint / prettier+eslint --fix.
- Single test: `node_modules/.bin/jest src/path/to/file.spec.ts -t "test name"`. CI uses `jest --runInBand`.

### Test environment flags

Many integration tests in [src/testing/integration/](src/testing/integration/) gate themselves on env vars (see [requires-linux-containers.ts](src/testing/requires-linux-containers.ts), [requires-windows-containers.ts](src/testing/requires-windows-containers.ts), [requires-kubernetes.ts](src/testing/requires-kubernetes.ts)). In CI, when the flag is unset the test is replaced with a no-op:

- `LINUX_CONTAINERS=true` — enable docker/linux-container tests (set on Linux + macOS in CI)
- `WINDOWS_CONTAINERS=true` — enable windows-container tests
- `CLUSTER_NAME=<ctx>` — enable kubernetes tests (CI uses minikube)
- `VERBOSE=true` — verbose test output

Locally these gates are bypassed (`isCI` is false), so tests will attempt to talk to Docker/k8s and fail if those aren't available. Use the env flags to opt-in selectively.

## Architecture

The CLI flows roughly: **parse → plan → schedule → execute**. Knowing this pipeline is the fastest way to orient in the codebase.

### Entry points

- [src/index.ts](src/index.ts) — bin entry, builds an `Environment` (cwd, file ctx, console, stdout/stderr, abortCtrl) and calls `runProgram`.
- [src/run-program.ts](src/run-program.ts) → [src/program.ts](src/program.ts) — commander CLI definition. Subcommands: `ls`, `exec`, `up`, `down`, `clean`, `store`, `restore`, `validate`, `package`. Each subcommand builds a `Cli` via `createCli(fileName, environment, workScope)`.
- [src/cli.ts](src/cli.ts) — the programmatic façade (`Cli` class) wrapping a `WorkTree` + `Environment`. Library consumers use `getCli()`; the CLI in `program.ts` is just a thin shell around this.

### Parse: `schema/` and `parser/`

- [src/schema/](src/schema/) — zod schemas for the build file (`build-file-*-schema.ts`) plus the parse pipeline: `schema-parser.ts` → `reference-parser.ts` → `work-scope-parser.ts`. `parse-error.ts` carries zod errors back to the CLI for pretty-printing.
- [src/parser/](src/parser/) — `read-build-file.ts`, `read-env-file.ts`, `default-build-file.ts`, label parsing. JSON schema published at [build.schema.json](build.schema.json) is generated from these zod schemas (see `src/json-schema.ts`).

### Plan: `planner/`

Parsing produces a **WorkTree** ([src/planner/work-tree.ts](src/planner/work-tree.ts)) of `WorkItem`s — `WorkTask` and `WorkService` indexed by id. `work-cache-id.ts` computes the per-task cache hash from sources + command + needs + image; this is what makes incremental builds work. Validation lives in `validate.ts` and `work-item-validation.ts`.

### Runtime selection

Each task/service is bound to a runtime: `local`, `docker`, or `kubernetes` (see `work-runtime-*.ts` in `planner/`, plus [src/runtime/runtime.ts](src/runtime/runtime.ts)). The runtime decides where execution happens.

### Schedule + execute: `executer/`

- [src/executer/execute-work-tree.ts](src/executer/execute-work-tree.ts) is the top-level loop. It builds a `ProcessManager` (worker pool), walks the work tree, and dispatches items.
- `scheduler/` holds the state machines (`task-state.ts`, `service-state.ts`, `dependency-state.ts`, `need-state.ts`) and cycle detection (`check-for-loop.ts`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [no0dles/hammerkit](https://github.com/no0dles/hammerkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
