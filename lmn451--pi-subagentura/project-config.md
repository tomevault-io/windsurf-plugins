---
trigger: always_on
description: A public [Pi](https://pi.dev) extension that adds in-process and attachable sub-agent tools. This file is for AI coding agents (and humans) working on the codebase.
---

# pi-subagentura — Agent Guidelines

A public [Pi](https://pi.dev) extension that adds in-process and attachable sub-agent tools. This file is for AI coding agents (and humans) working on the codebase.

## What this project is

- **npm package** `pi-subagentura` — published via OIDC trusted publishing on push of a `v*` tag.
- **Pi extension** — single entry point: `./src/subagent.ts` (declared in `package.json#pi.extensions`).
- **TypeScript, ESM, strict mode**, `target: ESNext`, Node ≥ 18, Pi SDK ≥ 0.80.6. CI verifies both the minimum and latest published Pi SDKs.
- **Runtime deps** are minimal: `ndjson`, `is-path-inside`. Pi SDKs are peer dependencies.
- **Tests** are `vitest` and live in `tests/` as `*.test.ts` (27 test files, ~12k lines of test code).
- **CI** is a single GitHub Actions workflow: typecheck → tests → published-tarball smoke → pack dry-run.

## Build / test / verify

Always run all of these before committing:

```bash
npm run typecheck   # tsc --noEmit, catches TDZ / no-use-before-define
npm test            # vitest run, 344+ tests
npm run format:check  # prettier --check .
npm run pack:check  # npm pack --dry-run, mirrors the publish step
```

The pre-commit hook (`simple-git-hooks` → `lint-staged` → `prettier --write`) formats staged files. The pre-push hook runs the third command across the repository. Install or refresh both with `npm run hooks:install`. Skip either for emergencies with `SKIP_SIMPLE_GIT_HOOKS=1`.

## Source layout (the 30-second tour)

| File                           | Purpose                                                                                                                                                                                                           |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `src/subagent.ts`              | **Entrypoint/barrel.** ~100 LOC. Default export registers all tool groups and session handlers; re-exports internals for test access.                                                                             |
| `src/tools/in-process.ts`      | `subagent_with_context`, `subagent_isolated`, async job management tools (`get_subagent_status`, `get_subagent_result`, `cancel_subagent`, `prune_subagent_jobs`), and `list_available_models`.                   |
| `src/tools/interactive.ts`     | Interactive sub-agent tools (`subagent_interactive`, `get_interactive_subagent_status`, `cancel_interactive_subagent`, `send_interactive_subagent_message`, `list_subagent_artifacts`, `read_subagent_artifact`). |
| `src/session-handlers.ts`      | `session_start`/`session_shutdown` handlers; poller interval setup/teardown on extension load/reload/shutdown.                                                                                                    |
| `src/artifact-poller.ts`       | Per-tick byte-ordered artifact walk, legacy session-JSONL tail-reading, durable delivery enqueue, and UI activity updates.                                                                                        |
| `src/rehydrate.ts`             | Reconstruct persisted cursors, queues, and delivery receipts on session start/reload/resume.                                                                                                                      |
| `src/helpers.ts`               | `startSubagentJob` primitive (in-process sub-agent runner), `resolveModel`, `formatUsage`, job registry and cleanup.                                                                                              |
| `src/artifact.ts`              | Versioned artifact protocol, immutable `outputs/<eventId>.md`, byte readers, mixed-v1 compatibility, and state-v2 helpers.                                                                                        |
| `src/child-protocol.ts`        | Child-only Pi lifecycle hooks selected by `PI_SUBAGENTURA_CHILD=1`.                                                                                                                                               |
| `src/delivery.ts`              | Bounded durable trigger-aware delivery queue and deterministic delivery IDs.                                                                                                                                      |
| `src/interactive-tmux.ts`      | `InteractiveSubagentState` and registry, launch-script builder, mux backend dispatch (is-alive, send-keys, kill-pane).                                                                                            |
| `src/multiplexer*.ts`          | Pluggable multiplexer interface + tmux and zellij backends. Registry auto-detects available backend at runtime.                                                                                                   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lmn451/pi-subagentura](https://github.com/lmn451/pi-subagentura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
