---
trigger: always_on
description: `ya-mono` is a workspace-first monorepo managed with `uv`.
---

## Repository Overview

`ya-mono` is a workspace-first monorepo managed with `uv`.

Workspace members:

- `packages/ya-agent-sdk` — SDK for building AI agents with Pydantic AI
- `packages/yaacli` — TUI reference implementation built on top of the SDK
- `packages/ya-claw` — workspace-native single-node runtime web service with `WorkspaceProvider`, in-process runtime state, schedules, bridges, and SQLite-first storage
- `packages/ya-agent-platform` — WIP stateless agent service with TBD scope

Shared repository areas:

- `apps/` — frontend applications and user-facing shells
- `skills/` — canonical skill sources and reference material
- `examples/` — runnable SDK examples
- `scripts/` — repository automation scripts
- `.github/` — CI and release workflows
- `Dockerfile.ya-claw` — YA Claw image build
- `Dockerfile.ya-claw-workspace` — official YA Claw Docker workspace image build
- `Dockerfile.ya-agent-platform` — YA Agent Platform image build
- `.dockerignore` — Docker build context rules

## Primary Package Focus

Most architecture work in this repository targets `packages/ya-agent-sdk` and `packages/ya-claw`.

- **Language**: Python 3.11+
- **Package Manager**: uv
- **Build System**: hatchling
- **Frontend Stack**: Vite + React + TypeScript

## Package Directions

### `packages/ya-agent-sdk`

- SDK for building AI agents with Pydantic AI
- preserves the core execution primitives used across the repository
- changes here should keep examples, skills, and package docs aligned

### `packages/yaacli`

- TUI reference implementation built on top of `ya-agent-sdk`
- runtime-facing CLI behavior belongs here

### `packages/ya-claw`

- active runtime product in this repository
- current delivery target is a single-node runtime
- `WorkspaceProvider` is the core extension boundary
- active session state, live events, async task coordination, schedules, and bridge coordination stay in process
- SQLite is the default durable store
- PostgreSQL is an optional durable store for deployments that prefer an external database
- local filesystem stores committed session continuity data
- requires `YA_CLAW_API_TOKEN` before service startup
- defaults: SQLite at `~/.ya-claw/ya_claw.sqlite3`, runtime data at `~/.ya-claw/data`, workspace root at `~/.ya-claw/workspace`, Docker workspace image `ghcr.io/wh1isper/ya-claw-workspace:latest`
- implementation style: organize runtime code by `api/`, `controller/`, and `orm/`
- internal data objects use Pydantic `BaseModel`
- code prefers explicit typing and `isinstance` checks
- session API is the high-level surface and run API is the low-level surface
- session metadata lives in the database
- committed continuity blobs live in `run-store/{run_id}/state.json` and `run-store/{run_id}/message.json`
- `message.json` stores the compacted replay list of AGUI-aligned events as a top-level JSON array
- session GET exposes paginated runs with optional raw `input_parts` and compacted message replay lists, returns optional top-level committed state/message from `head_success_run_id`, and derives session status from the latest run
- session turns API returns successful completed turns with raw `input_parts`, `output_text`, and `output_summary`
- run GET returns `session + run + optional state + optional message`; run trace API returns compact tool-call/tool-response projections from `message.json`
- built-in `session` toolset lets agents inspect only their current session via internal HTTP client tools `list_session_turns` and `get_run_trace`; session ID and bearer token stay inside the client resource
- runtime instance heartbeat lives in `runtime_instances`; run records carry claim ownership through `claimed_by` and `claimed_at`
- rerun can explicitly target failed or interrupted runs through `restore_from_run_id`
- input payloads use `input_parts` rather than a single `input_text`; run records preserve `input_parts` as original JSON-compatible payloads for replay/UI reconstruction
- successful run records store final `output_text` directly in the database and keep `output_summary` for compact displays
- foundational execution modules live under `ya_claw/execution/`
- workspace provider modules live under `ya_claw/workspace/`
- `LocalWorkspaceProvider` uses `LocalFileOperator` plus `LocalShell` over the real workspace path
- `DockerWorkspaceProvider` uses Docker mounts through `SandboxEnvironment`; file operations map the service-visible workspace path to `/workspace`, and Docker shell uses `/workspace`
- `YA_CLAW_WORKSPACE_PROVIDER_DOCKER_HOST_WORKSPACE_DIR` provides the Docker daemon-visible host mount path when the YA Claw service itself runs in Docker
- Docker workspace containers receive UID/GID envs (`YA_CLAW_WORKSPACE_UID`, `YA_CLAW_WORKSPACE_GID`, `YA_CLAW_HOST_UID`, `YA_CLAW_HOST_GID`) from the service process by default or from `YA_CLAW_WORKSPACE_PROVIDER_DOCKER_UID/GID`
- `Dockerfile.ya-claw` can drop service execution privileges through `YA_CLAW_RUN_UID` and `YA_CLAW_RUN_GID`; the official workspace image defaults to UID/GID 1000 through build args
- built-in run orchestration lives in `ya_claw/execution/coordinator.py`
- built-in coordinator dispatch resolves model/runtime behavior from AgentProfile rows; `YA_CLAW_DEFAULT_PROFILE` defaults to `default`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Wh1isper/ya-mono](https://github.com/Wh1isper/ya-mono) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
