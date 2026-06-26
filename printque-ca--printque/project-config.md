---
trigger: always_on
description: - All code changes must be backwards compatible and non-destructive to existing user data and setup; migrations should preserve or recover data, not silently wipe it.
---

## Learned User Preferences

- All code changes must be backwards compatible and non-destructive to existing user data and setup; migrations should preserve or recover data, not silently wipe it.
- Tests must never read or write live user data under `~/PrintQueData`; the app and pytest should be safe to run at the same time.
- When implementing an attached plan: do not edit the plan file; use the existing todos (do not recreate them); mark each in progress and finish all before stopping.
- Prefer concise, direct answers when asked what changed or how something works.
- When fixing lint/format issues, run the linter and fix what it reports directly instead of many exploratory diagnostic commands.
- Queue quantity and delete should use optimistic client updates with debounced API sync so rapid clicks feel instant without waiting for the next poll.
- Do not manually bump version in `api/__version__.py` or `pyproject.toml`; mark breaking changes with `feat!:` / `BREAKING CHANGE:` and let CI semantic-release update version on merge to `main`.

## Learned Workspace Facts

- Monorepo layout: Python Flask API in `api/`, React/Vite UI in `app/`. Local dev from repo root: `pnpm run dev` (UI) and `pnpm run api` (Flask) via root `package.json` scripts. On Windows, stopping the terminal may leave prior `python app.py` orphans running — kill them before another `pnpm run api` to avoid duplicate background threads and stale `printers_rwlock` logs. Frontend Biome config in `app/biome.json`; repo-root `.vscode/settings.json` with Biome format-on-save and `biome.configurationPath: app/biome.json`. Do not gitignore all `*.json` — exclude only root runtime state filenames.
- Persistent user data lives under `{DATA_DIR or home}/PrintQueData/` via `api/utils/paths.py` (JSON state files, `uploads/`, logs, migration backups).
- Backend uses Flask-SocketIO with `async_mode='threading'`; background threads that emit Socket.IO events should use `utils/socketio_emit.py` (`emit_status_update` / `safe_emit` with app context). Never emit while holding `WriteLock(printers_rwlock)` — it deadlocks read paths (`api_get_printers`, monitors). Avoid `increment_queue_sent_count` and queue disk writes inside that write lock; `status_poller` defers them until after the write section. On API restart, status poller preserves persisted printer state until live status is confirmed (`BAMBU_LIVE_STATUS_SEEN` / `PRUSA_LIVE_STATUS_SEEN`); first poll batch waits `STARTUP_POLL_DELAY_SECONDS` (5s). OS-thread workers and native locks: `utils/threading_compat.py`, `spawn_os_thread` in `printer_utils.py`.
- Global `enhanced_prusa_ejection_monitoring` thread (~10s) is named "Prusa ejection monitor"; it skips `type == 'bambu'` (missing `type` defaults to `prusa`). Its log errors are usually `printers_rwlock` read timeouts, not Prusa ejection running on Bambu.
- Library catalog persists in `library.json`; print queue jobs in `queue.json` (`QUEUE_JOBS`). `sent`/`quantity` survive API restart; startup does not run distribution. Distributor walks the queue front-to-back (`sent < quantity`); jobs snapshot library metadata and reference ejection presets by `ejection_code_id` (G-code resolved at runtime). On Bambu, `sent` increments only on PREPARING/PREPARE→PRINTING (not FTP upload, pause/resume, or OFFLINE/ERROR recovery); `count_incremented_for_current_job` is persisted after increment and must not be cleared on transient OFFLINE/idle updates. `periodic_pending_distribution_check` (PendingDistribution thread, every `Config.DISTRIBUTION_INTERVAL` / 30s) retriggers distribution when partial jobs and READY/IDLE printers coexist — fixes event-only dead states (e.g. Ready + 1/N with no error). Queue jobs may store `last_error` / `error_events` via `record_queue_job_error` in `library_queue.py`. Legacy `orders.json` is migrated on load. Queue delete is soft-delete by queue job `id` (`DELETE /api/v1/queue/<id>`); `GET /queue` omits `deleted: true`, but Socket.IO `status_update` may push the full `QUEUE_JOBS` list (including deleted rows) and briefly undo optimistic UI deletes until the next poll.
- Dashboard/stats via `GET /api/v1/system/stats`: `total_filament` is kilograms (do not divide by 1000 in the UI); `queue_pending_count` = jobs waiting (`sent === 0`); legacy `in_queue_count` = partial multi-copy only (`0 < sent < quantity`); `active_prints` = printers in PRINTING; `completed_today` counts fulfilled queue jobs when the last copy is counted (at `sent` increment), not when the print physically finishes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PrintQue-ca/PrintQue](https://github.com/PrintQue-ca/PrintQue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
