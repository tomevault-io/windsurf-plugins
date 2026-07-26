---
trigger: always_on
description: Last updated: 2026-07-14 (codebase-scanned; concise agent-focused)
---


# User Space Feature Implementation Instructions

Last updated: 2026-07-14 (codebase-scanned; concise agent-focused)

## Scope

- Apply to User Space runtime/collab/share work in `ragtime/userspace/**` and `runtime/**`.
- Keep changes incremental; do not invent new UX or role semantics.
- Endpoint details belong in `/docs` (do not maintain static route dumps here).

## Big Picture (How User Space Actually Works)

- Control plane is in Ragtime app code (`ragtime/userspace/runtime_service.py`, `ragtime/userspace/runtime_routes.py`).
- Data plane runs in runtime manager/worker code (`runtime/manager/**`, `runtime/worker/**`).
- `runtime/main.py` chooses service mode via `RUNTIME_SERVICE_MODE`; manager mode also mounts worker routes.
- Workspace runtime files live under `${INDEX_DATA_PATH}/_userspace/workspaces/{workspace_id}/files`.
- Preview is runtime-only and now uses dedicated per-workspace preview origins dispatched by `PreviewHostDispatchMiddleware` in `ragtime/userspace/preview_host.py`.
- Preview launch is an explicit API step: control-plane routes mint short-lived bootstrap grants and the preview host mints a preview session cookie on first load.
- Browser-auth cookie surfaces are only for `collab` and `runtime_pty`; preview no longer uses capability cookies.

## Workspace Code Index Queue

- Hidden User Space code indexes run through `WorkspaceCodeIndexService` (`ragtime/userspace/workspace_code_index_service.py`), not the generic filesystem indexer.
- Dirty paths, manual reindex, missing-baseline reconcile, and `process_dirty_workspace()` all converge on durable `workspace_code_index_jobs` rows. Do not reintroduce direct `asyncio.create_task()` indexing paths.
- Queue state is DB-owned: `pending` rows carry `waiting_for_job_id`; the runner atomically claims rows with `FOR UPDATE SKIP LOCKED`, clears the wait pointer, then runs them as `indexing`.
- `userspace_code_index_max_concurrency` controls active jobs (default `1`, admin range `1..8`), but one workspace must not have two active code-index jobs at once.
- Pending jobs cancel immediately; running jobs set `cancel_requested` and trip the in-memory cancellation flag checked during processing. Keep `cancelled` distinct from `failed`.

## Share Routing Split (Critical)

- Public share URLs are top-level routes in `ragtime/main.py`:
  - `/{owner_username}/{share_slug}` (canonical)
  - `/shared/{share_token}` (anonymous/token form)
- Public share routes do not proxy app bytes directly anymore; they validate access, then redirect to a dedicated preview origin bootstrap URL.
- Internal editor preview APIs are launch endpoints in `ragtime/userspace/runtime_routes.py` under `/indexes/userspace/runtime/workspaces/.../preview-launch`, and shared launches live under `/indexes/userspace/shared/.../preview-launch`.
- Public and internal routes are still different paths with different auth contexts; when changing share behavior, update both layers intentionally.
- Public password-protected shares use FastAPI-rendered full-page HTML prompt + scoped cookie (`userspace_share_pw_*`) in `main.py`.

## Runtime + Bootstrap Conventions

- Runtime bootstrap config is workspace-local: `.ragtime/runtime-bootstrap.json`.
- Bootstrap execution stamp is `.ragtime/.runtime-bootstrap.done` (`runtime/core/shared.py`).
- Default bootstrap template is managed in `ragtime/userspace/service.py` (`_default_runtime_bootstrap_config`, template version 5). Auto-update logic includes `_is_legacy_default_bootstrap` detection for configs missing `managed_by`/`template_version`.
- Worker startup reads bootstrap config and reruns commands when config digest changes (`runtime/worker/service.py`). Digest includes file content of `watch_paths`.
- Bootstrap retry: if devserver exits with "command not found" patterns, the worker invalidates the stamp and retries on next start (`_should_retry_bootstrap_after_exit`).
- Runtime launch is defined by `.ragtime/runtime-entrypoint.json` (command/cwd/framework) consumed by worker service.
- `.ragtime/runtime-entrypoint.json` is authoritative. Runtime no longer falls back to `package.json`, Python entrypoint guesses, or `index.html` heuristics.
- Keep launch commands `$PORT`-aware and bound to `0.0.0.0` for proxy reachability.
- SQLite persistence uses `.ragtime/db/migrations/*.sql` as source-of-truth schema history and `.ragtime/scripts/sqlite_migrate.py` as the default runner scaffold.
- The runtime bootstrap template includes a migration apply command for the default runner and watches both the runner path and migrations directory for digest changes.
- Agents may update `.ragtime/scripts/sqlite_migrate.py` and `.ragtime/db/migrations/*` via file tools when implementing persistence changes.
- `dashboard_entrypoint.js`, `runtime_bridge.js`, and `sqlite_migrate.py` templates live in `templates/`.
- Independent preview subdomains affect routing, cookies, and origin isolation only; they do **not** make package/bare imports invalid by themselves.
- Node/server files and bundled frontend apps can legitimately import dependencies declared in `package.json` (and Node built-ins where applicable). Only browser-served source modules need browser-resolvable specifiers at runtime.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mattv8/ragtime](https://github.com/mattv8/ragtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
