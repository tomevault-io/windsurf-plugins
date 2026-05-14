---
trigger: always_on
description: - `ada_backend/` — API server (FastAPI). Routers in `routers/`, services in `services/`, repos in `repositories/`.
---


# Architecture Rules

## Architecture Overview

### Repo Structure

- `ada_backend/` — API server (FastAPI). Routers in `routers/`, services in `services/`, repos in `repositories/`.
- `engine/` — Graph execution engine. `graph_runner/` (DAG scheduler), `components/` (component implementations), `field_expressions/` (AST).
- `mcp_server/` — Standalone MCP server wrapping the API. Separate process, no imports from `ada_backend/` or `engine/`.
- `data_ingestion/` — Document ingestion logic.
- `workers/` — Ingestion + webhook worker processes.
- `infra/k8s/` — Kustomize-based Kubernetes manifests.
- `tests/` — Test suite.

### Backend Pattern

Router → Service → Repository (SQLAlchemy + PostgreSQL). Routers define HTTP endpoints and auth dependencies. Services encapsulate business logic. Repositories handle DB CRUD.

### Auth Split

Supabase owns users, organizations, memberships, roles. Backend owns projects, graphs, runs, components, and everything else. Backend validates Supabase JWTs via `supabase.auth.get_user()`. Organization roles are checked via Supabase Edge Functions (`check-org-access`, `check-super-admin`).

### API Keys

SHA-256 hashed (salted with `BACKEND_SECRET_KEY`), prefix `taylor_`, scoped to project or organization.

### Graph Execution

DAG of ComponentInstances connected via FieldExpressions (JSON AST). See `ada_backend/docs/engine.md`.

### Environments

`draft` / `production` bound to specific GraphRunner versions via `ProjectEnvironmentBinding`. Each `Run` records its `graph_runner_id` (set when the worker transitions the run to `RUNNING`).

### Project Tags

Free-form string tags on projects via `project_tags` table (many-to-many). Tags are lowercase, unique per project. Used for filtering/organizing projects in the UI. Git-synced projects automatically receive a `"github"` tag. Management endpoints: `POST /projects/{id}/tags`, `DELETE /projects/{id}/tags/{tag}`. List endpoint supports `?tags=foo&tags=bar` filter (AND semantics). Org-level autocomplete: `GET /projects/org/{org_id}/tags`.

### Workers

- Queue workers share a `BaseQueueWorker` ABC in `ada_backend/workers/base_queue_worker.py` (heartbeat, per-worker processing list, orphan recovery with capped follow-up scans, drain). Orphan recovery runs at startup plus up to `_MAX_ORPHAN_FOLLOW_UPS` (2) follow-up scans spaced by `_ORPHAN_FOLLOW_UP_DELAY` (= heartbeat TTL) to catch dead workers whose heartbeat hadn't expired during the startup scan; after that, no more scans run. On graceful shutdown the heartbeat key is deleted eagerly before returning items to the main queue. Concrete subclasses only implement payload processing and entity recovery.
- Run queue: `RunQueueWorker` in `ada_backend/workers/run_queue_worker.py`, daemon thread in API process. Run input data is durably persisted in the `run_inputs` table (keyed by `retry_group_id` with a unique constraint) so async processing can recover canonical input from Postgres instead of relying only on Redis payloads. New runs initialize a dedicated `retry_group_id` on first attempt (distinct from `run.id`), and retries keep that same group id. `run_inputs.created_at` is non-null and indexed for retention cleanup scans.
- QA queue: `QAQueueWorker` in `ada_backend/workers/qa_queue_worker.py`, daemon thread in API process. See `ada_backend/docs/qa-system.md`.
- Git sync queue: `GitSyncQueueWorker` in `ada_backend/workers/git_sync_queue_worker.py`, daemon thread in API process. Processes two payload types on the same queue: graph sync jobs (`sync_graph_from_github`) and prompt sync jobs (`sync_prompts_from_github`). Graph sync deploys reuse the same deploy service path as frontend publish (production promotion + fresh draft clone). Prompt sync creates new `PromptVersion` rows for changed `.md` files under `draftnrun/prompts/`. See `ada_backend/docs/git-sync.md`.
- Webhook worker: Redis Stream consumer, separate process. Three patterns: provider webhooks (external → backend), user-triggered, internal (worker → API). Provider-triggered executions create runs and enqueue to the run queue; if enqueue fails after run creation, webhook service marks the run `FAILED` immediately to avoid orphan `PENDING` runs. Direct triggers persist input to `run_inputs` before webhook-stream enqueue, then call the internal run endpoint which enqueues the run to the durable Redis run queue (`RunQueueWorker`) for heartbeat-based orphan recovery. Scheduler-triggered internal runs must always include `cron_id` (job id) in the internal webhook body so API-side tracing can keep the cron correlation after the process boundary; include `cron_run_id` (execution id) only for the top-level scheduler-owned execution that updates CronRun state. The `RunQueueWorker` handles both `cron_id` (tracing) and `cron_run_id` (CronRun status transitions: QUEUED→RUNNING→COMPLETED/ERROR). Endpoint-polling fan-out runs forward `cron_id` without reusing the parent `cron_run_id`. Stream ACK is outcome-based: retryable failures stay pending (no ACK), success and fatal outcomes are ACKed, with dead-letter after `MAX_DELIVERY_ATTEMPTS`. See `ada_backend/docs/webhooks.md`.
- Ingestion worker: Redis Stream consumer, separate process.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Scopeo/draftnrun](https://github.com/Scopeo/draftnrun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
