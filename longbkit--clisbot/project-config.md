---
trigger: always_on
description: These rules apply to everything inside this repository.
---

# AGENTS.md

## Scope
These rules apply to everything inside this repository.

The stable implementation contract lives in:
- `docs/architecture/architecture-overview.md`
- `docs/architecture/surface-architecture.md`
- `docs/architecture/runtime-architecture.md`
- `docs/architecture/model-taxonomy-and-boundaries.md`

If implementation conflicts with those docs:
1. stop
2. refactor toward the docs if the fix is clear
3. ask the user before proceeding if the conflict changes behavior, architecture, or scope

Do not silently drift away from the architecture docs.

If you are asked to code in any repo and that repo or one of its subfolders has `AGENTS.md`, `CLAUDE.md`, or `GEMINI.md`, pick the one that applies to the current folder scope and follow it strictly.

## First Read Order
Load context in this order unless the task is obviously narrower:
1. `README.md`
2. `docs/overview/README.md`
3. the architecture docs listed above

Then load only the smallest extra context that matches the task:
- feature work: `docs/features/README.md`, the relevant feature doc, then linked task docs
- task execution: `docs/tasks/README.md`, `docs/tasks/backlog.md`, then the relevant task doc
- operator or onboarding or release work: `docs/development/README.md` and the relevant user-guide doc
- research-heavy questions: the relevant file under `docs/research/`

Do not front-load the whole docs tree.

## Documentation Precedence
When documents disagree, use this order:
1. `docs/architecture/`
2. `README.md`, `docs/development/README.md`, and `docs/user-guide/`
3. `docs/features/`
4. `docs/tasks/`
5. `docs/research/` and `docs/lessons/`

`docs/research/` and `docs/lessons/` are supporting context only. They should not silently override architecture or guide docs.

## Repo Map
Use this ownership map before editing:
- `src/channels`: Slack and Telegram surfaces, route handling, rendering, pairing, follow-up behavior
- `src/agents`: durable agent state, sessions, queueing, loops, attachments, run lifecycle
- `src/auth`: roles, permissions, owner claim, authorization resolution
- `src/config`: schema, loading, credentials, templates
- `src/control`: operator CLI, runtime lifecycle, health, status, logs, bootstrap
- `src/runners`: execution backends, currently tmux
- `src/shared`: cross-cutting utilities
- `test/`: regression and behavior coverage
- `docs/tests/`: readable validation scenarios when behavior needs explicit ground truth

## Command Baseline
Prefer these repo-standard commands:
- install: `bun install`
- typecheck: `bunx tsc --noEmit`
- targeted tests: `bun test <file>`
- full tests: `bun test`
- full local gate: `bun run check`
- local dev start: `bun run start ...`
- local dev status: `bun run status`
- local dev logs: `bun run logs`
- package build: `bun run build`
- publish: run `npm login`, then run exactly `npm publish --access public`
- when `npm login`, `npm publish --access public`, or another external auth command returns a browser link, keep the process open, send the link to the user, wait for them to finish auth, then continue the same flow
- do not switch publish auth into a manual `--otp` handoff; keep the normal interactive publish flow and hand off only the exact link or prompt the tool returns from that same live process

Do not invent ad hoc verification flows when one of these commands already fits.

## Runtime And Env Precedence
Repo-local convenience scripts use the repo `.env` and default to:
- `CLISBOT_HOME=~/.clisbot-dev`

Treat runtime path resolution in this order:
1. explicit CLI or function parameters
2. explicit `CLISBOT_*` path env vars
3. `CLISBOT_HOME`-derived defaults

If runtime behavior looks wrong, inspect `CLISBOT_HOME`, `CLISBOT_CONFIG_PATH`, `CLISBOT_PID_PATH`, `CLISBOT_LOG_PATH`, and related runtime env vars before assuming the code is wrong.

## Documentation Workflow
Use the repo doc systems consistently:
- `docs/overview/README.md` is the top-level project overview
- `docs/overview/human-requirements.md` is raw human input; do not edit it unless the user explicitly asks
- `docs/tasks/backlog.md` is the source of truth for task status and priority
- `docs/features/feature-tables.md` is the source of truth for feature state
- `docs/research/<feature>/` is for source-driven analysis that is not yet stable contract
- `docs/features/non-functionals/` is for cross-cutting quality work
- `docs/lessons/` is for reusable lessons from repeated feedback, delivery struggles, or durable operator preferences
- keep task docs brief when they mostly track research work; link to `docs/research/` instead of duplicating analysis
- keep task docs in the task workflow and feature docs in the feature workflow

Prefer links over repeated context.

## Design Defaults
Follow these defaults unless the user explicitly asks for a different tradeoff:
- KISS: prefer the smallest change that keeps architecture, runtime truthfulness, and operator flow clear
- DRY: prefer one shared implementation path over parallel wrappers or duplicated mutations
- backend-facing models must stay resource-oriented and revision-aware
- do not leak transient runtime state into persistence contracts
- use `docs/architecture/model-taxonomy-and-boundaries.md` for model naming, ownership, lifecycle, and mapping boundaries

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [longbkit/clisbot](https://github.com/longbkit/clisbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
