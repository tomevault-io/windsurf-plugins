---
trigger: always_on
description: Context and conventions for AI assistants working on the Optio codebase.
---

# CLAUDE.md

Context and conventions for AI assistants working on the Optio codebase.

## What is Optio?

Optio is an orchestration system for AI coding agents. Think of it as "CI/CD where the build step is an AI agent." One primary user-facing concept (Tasks) with one attribute (has a repo) flipping the pipeline, plus shared primitives:

**Tasks** — a configured unit of agent work. A Task has a **Who** (agent type), **What** (prompt or template), **When** (trigger: manual / schedule / webhook / ticket), optional **Where** (repo + branch), and **Why** (description). Tasks come in two flavors based on whether a repo is attached:

- **Repo Task** — `Where` is set. The agent clones the repo into a worktree and opens a PR:
  1. Spins up an isolated Kubernetes pod for the repository (pod-per-repo)
  2. Creates a git worktree for the task (multiple run concurrently per repo)
  3. Runs Claude Code, OpenAI Codex, GitHub Copilot, Google Gemini, or OpenCode with the prompt
  4. Streams structured logs back to a web UI in real time
  5. Agent stops after opening a PR (no CI blocking)
  6. PR watcher tracks CI checks, review status, and merge state
  7. Auto-triggers code review agent on CI pass or PR open (if enabled)
  8. Auto-resumes agent when reviewer requests changes (if enabled)
  9. Auto-completes on merge, auto-fails on close

- **Standalone Task** — no `Where`. The agent runs in an isolated pod with no repo checkout, producing logs and side effects (e.g., queries Slack, posts to a database). Scheduled/webhook-driven runs of this flavor are the common case.

**Scheduled (Task Configs)** — a saved Task blueprint that spawns fresh Tasks on a trigger firing. Stored in `task_configs`. Each firing calls `instantiateTask()` which goes through the full Repo Task pipeline. Manageable at `/tasks/scheduled`. Standalone equivalents are stored in `workflows` (see backend-naming note below).

**Triggers** — polymorphic table `workflow_triggers` keyed by `(target_type, target_id)`. `target_type` is `"job"` (Standalone Tasks) or `"task_config"` (Repo Tasks). Trigger types: `manual`, `schedule` (cron), `webhook`, `ticket`. The `workflow-trigger-worker` polls due schedule triggers and dispatches to the correct target service.

**Templates** — reusable prompt templates in `prompt_templates` with a `kind` discriminator (`prompt` / `review` / `job` / `task`). Supports `{{param}}` substitution and `{{#if param}}...{{/if}}` blocks. Rendered lazily on trigger firing so params from the trigger payload substitute into the prompt.

**Connections** — external service integrations injected into agent pods at runtime via MCP (Model Context Protocol). Built-in providers: Notion, GitHub, Slack, Linear, PostgreSQL, Sentry, Filesystem. Also supports custom MCP servers and HTTP APIs. Fine-grained access control (per-repo, per-agent-type, permission levels).

**Backend-naming note.** For historical reasons the tables are `tasks` (Repo Tasks' one-time runs), `task_configs` (Repo Task blueprints), and `workflows` / `workflow_runs` / `workflow_triggers` (Standalone Tasks and their shared trigger surface). User-facing copy never uses "Workflow" or "Job" — everything is "Task" / "Repo Task" / "Standalone Task" in the UI. The legacy `/api/workflows` path was renamed to `/api/jobs` at the HTTP layer and the `/jobs/*` web routes remain for Standalone detail/runs.

For the long-form explanation of how the two flavors map to the three internal types, the polymorphic HTTP layer, and how the UI presents them, see `docs/tasks.md`.

**Unified `/api/tasks` HTTP layer.** All three kinds (`repo-task`, `repo-blueprint`, `standalone`) are reachable through one polymorphic HTTP resource:

- `GET /api/tasks?type=repo-task|repo-blueprint|standalone|all` — unified list
- `POST /api/tasks` — body takes `{ type, ... }`; dispatches to taskService, taskConfigService, or workflowService based on type
- `GET /api/tasks/:id` — resolves the id across all three tables; returns native row tagged with `type` discriminator
- `GET/POST /api/tasks/:id/runs[/:runId]` — polymorphic runs (spawned `tasks` for blueprints, `workflow_runs` for standalone, 405 for ad-hoc)
- `GET/POST/PATCH/DELETE /api/tasks/:id/triggers[/:triggerId]` — polymorphic triggers (405 for ad-hoc repo-task)
- Resolver: `unified-task-service.resolveAnyTaskById()` checks tasks → task_configs → workflows; UUIDs are globally unique so no collision

Legacy `/api/jobs/*` and `/api/task-configs/*` endpoints still work as thin aliases for back-compat.

## Architecture

```
┌─────────────┐     ┌──────────────┐     ┌─────────────────────┐
│   Web UI    │────→│  API Server  │────→│   K8s Pods          │
│  Next.js    │     │   Fastify    │     │                     │
│  :30310     │     │   :30400     │     │  ┌─ Repo Pod A ──┐  │
│             │←ws──│              │     │  │ clone + sleep  │  │
│             │     │ - BullMQ     │     │  │ ├─ worktree 1  │  │
│             │     │ - Drizzle    │     │  │ ├─ worktree 2  │  │
│             │     │ - WebSocket  │     │  │ └─ worktree N  │  │
│             │     │ - PR Watcher │     │  └────────────────┘  │
│             │     │ - Workflow Q │     │  ┌─ Workflow Pod ─┐  │
│             │     │ - Health Mon │     │  │ isolated agent  │  │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonwiggins/optio](https://github.com/jonwiggins/optio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
