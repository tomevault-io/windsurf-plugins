---
trigger: always_on
description: A **Kopf-based Kubernetes operator** that deletes Deployments whose duration-based TTL annotation (`kubetimer.io/ttl`, e.g. `30m`, `2h`, `7d`) has expired. On annotation detection, the operator computes the expiry time (`now + duration`), stores it as a `kubetimer.io/expires-at` annotation (ISO 8601), and schedules an APScheduler `DateTrigger` job. Deletions are **event-driven** — not periodic polling. Configuration comes from `KUBETIMER_*` environment variables (Pydantic Settings). Python 3.14,
---

# KubeTimer – Copilot Instructions

## What This Is

A **Kopf-based Kubernetes operator** that deletes Deployments whose duration-based TTL annotation (`kubetimer.io/ttl`, e.g. `30m`, `2h`, `7d`) has expired. On annotation detection, the operator computes the expiry time (`now + duration`), stores it as a `kubetimer.io/expires-at` annotation (ISO 8601), and schedules an APScheduler `DateTrigger` job. Deletions are **event-driven** — not periodic polling. Configuration comes from `KUBETIMER_*` environment variables (Pydantic Settings). Python 3.14, managed with **uv** and **hatchling** build backend.

## Architecture & Data Flow

```
K8s watch events → Kopf event handlers → APScheduler DateTrigger jobs → K8s API delete
                                                                ↑
Startup: list API (paginated) → reconcile orchestrator → bulk_delete / schedule_deletion_job
```

### Key modules

- **`kubetimer/__init__.py`** – `register_all_handlers()` imperatively registers all Kopf handlers (`on.startup`, `on.cleanup`, `on.create`, `on.field`, `on.delete`). Also contains `startup_handler` (loads config, starts APScheduler, runs reconciliation) and `shutdown_handler`.
- **`handlers/deployment.py`** – Three thin event handlers: `on_deployment_created_with_ttl`, `on_ttl_annotation_changed`, `on_deployment_deleted_with_ttl`. They parse the duration TTL, compute expiry, PATCH the `kubetimer.io/expires-at` annotation, and delegate scheduling to `scheduler/jobs.py`.
- **`handlers/registry.py`** – Only `configure_memo()` — populates `kopf.Memo` from Settings.
- **`scheduler/jobs.py`** – APScheduler job management: `schedule_deletion_job` (DateTrigger), `cancel_deletion_job`, `delete_deployment_job` (re-verifies UID + TTL before deleting).
- **`reconcile/`** – Startup recovery: `orchestrator.py` fetches all TTL-annotated Deployments, triages into expired vs future, bulk-deletes expired ones. Uses `fetcher.py` (paginated list API, sync/async delete wrappers), `bulk_delete.py` (semaphore-bounded concurrent deletes), `models.py` (`TtlDeployment` frozen dataclass).
- **`config/settings.py`** – `Settings(BaseSettings)` with `KUBETIMER_` env prefix. `get_settings()` is `@lru_cache`-ed.
- **`config/k8s.py`** – K8s client loading, `@lru_cache`-ed `apps_v1_client()`, connection pool sizing.
- **`utils/time_utils.py`** – `parse_ttl_duration()` (duration string → timedelta), `parse_expires_at()` (ISO 8601 → datetime), `is_ttl_expired()`.
- **`utils/namespace.py`** – `should_scan_namespace()` with include/exclude filtering.
- **`utils/logs.py`** – structlog setup; `setup_logging()` is `@lru_cache`-ed.
- **`main.py`** – Sets up `uvloop`, calls `register_all_handlers()`, runs `kopf.run()`.

## Key Patterns

- **Event-driven, not polling** – Kopf watches trigger handlers on create/update/delete. APScheduler `DateTrigger` fires a one-shot job at the exact TTL expiry time. No periodic scanning.
- **`kopf.Memo` as shared state** – Runtime config (annotation_key, dry_run, timezone, namespace filters, scheduler, reconciling_uids) lives on `memo`. Populated by `configure_memo()` in `startup_handler`.
- **Imperative handler registration** – All `kopf.on.*` calls happen in `register_all_handlers()` (in `__init__.py`), not via top-level decorators, so they can reference module-level settings.
- **Reconciling UIDs guard** – `memo.reconciling_uids: set[str]` prevents event handlers from double-processing Deployments during startup reconciliation. UIDs are added before reconciliation, discarded as each job completes.
- **Re-verification before delete** – `delete_deployment_job` re-reads the Deployment from the API and checks UID match + TTL still expired before deleting. Two-tier expiry resolution: (1) prefer `kubetimer.io/expires-at` annotation, (2) fallback to `creation_timestamp + parse_ttl_duration(ttl)`. This guards against recreated resources or annotation changes.
- **Sync-in-async via `asyncio.to_thread`** – The kubernetes client is synchronous. All blocking API calls are wrapped with `asyncio.to_thread` to keep the event loop free.
- **Structured logging** – Use `structlog` with snake_case event names (e.g. `"deployment_deleted_by_scheduler"`). Get loggers via `get_logger(__name__)`.

## Dev Workflow

```bash
uv sync                        # install deps
make install                   # editable install (uv pip install -e .)
python kubetimer/main.py       # run locally (needs kubeconfig)
./deploy.sh                    # deploy to cluster (namespace, RBAC, Deployment)
make test                      # run unit tests (uv run --group dev pytest)
make coverage                  # pytest with coverage report
make check                     # format-check + lint + typecheck + test
make fix                       # auto-format + lint
```

## Testing Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kubetimer/kubetimer](https://github.com/kubetimer/kubetimer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
