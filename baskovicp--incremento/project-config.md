---
trigger: always_on
description: Compact repo guide for coding agents. Keep this file high-signal and current.
---

# Incremento Agent Notes

Compact repo guide for coding agents. Keep this file high-signal and current.

## Repo Shape

- `__init__.py`: addon entry point, hook registration, settings save/load, reviewer patches, bridge startup.
- `backend/`: scheduling, persistence, content managers, profile-aware paths, search, recovery, statistics, and the authenticated browser bridge.
- `frontend/`: Qt dialogs/docks, editor integrations, reviewer overlays, and React source/tests for the PDF viewer.
- `chrome_extensions/incremento_companion/`: Manifest V3 source, tests, static entry pages, and committed runtime bundles for imports, capture, bookmarks, and playback sync.
- `web/`: shipped reader/player assets. `web/dist/pdf_viewer.js` is generated from `frontend/src/`; `web/pdfjs/` is vendored PDF.js runtime code.
- `tests/`: Python unit, integration, real-Anki subprocess, packaging, repair-harness, and selected UI regression coverage.
- `scripts/`: release packaging, guarded repair/eval/smoke tooling, and extension icon generation.
- `.github/workflows/verify.yml`: supported-Python CI, deterministic repair evals, dependency audit, frontend/extension tests, and generated-asset drift checks.
- `pyproject.toml`, `pytest.ini`, `requirements-dev.txt`: incremental Ruff/mypy policy, pytest coverage defaults, and the reproducible development dependency set; focused commands may override `addopts`, but committed CI expectations remain authoritative.
- `.gitignore`: local/runtime/build exclusions. A path being ignored does not make it safe to package or inspect; packaging has its own explicit allowlist.
- `config.json`: shipped defaults only; runtime normalization and persistence belong to `backend/config_service.py`.
- `README.md`, `MANUAL.md`, `ARCHITECTURE.md`, `SECURITY.md`, `EXPORTING.md`: developer overview, user behavior, system boundaries, security policy, and backup/restore contract.
- `LICENSE`: distribution terms; keep it in release artifacts and do not rewrite or remove notices during dependency/vendor updates.
- `plan.drawio.xml`: design artifact only; it is not packaged runtime state.
- `user_files/`: runtime data only. All user data is per-profile.

Important newer hotspots:

- `backend/db_connection.py`, `backend/db_schema.py`, `backend/db.py`: per-thread/profile SQLite lifecycle, atomic schema ledger, legacy repository surface, and ordered migrations.
- `backend/operation_journal.py`, `backend/reconciliation.py`, `backend/migration.py`, `backend/note_metadata.py`: stable content identity, crash-safe cross-store imports, bounded profile-open recovery, explicit full reconciliation, and resumable legacy storage migration.
- `backend/config_service.py`, `config.json`: versioned config normalization and the canonical config read/write boundary.
- `backend/search_indexer.py`, `backend/search_repository.py`, `frontend/search_all.py`: cancellable off-main PDF indexing, optional FTS-backed bounded search, and the Search ALL read model.
- `backend/anki_compat.py`, `frontend/session_launcher.py`, `backend/session.py`: private Anki reviewer compatibility boundary and frontend-owned session launch UI.
- `backend/knowledge_tree.py`, `backend/knowledge_tree_postpone.py`, `frontend/knowledge_tree_dialog.py`: the knowledge-tree workspace, branch study, branch priority tools, postpone flow, and subset review.
- `backend/session.py`, `backend/scheduler_config.py`, `frontend/learn_dialog.py`: Incremento session construction, active-session auto-refill, and the scheduler dialog controls for card states and pending-window behavior.
- `frontend/session_setup_model.py`, `frontend/learn_dialog.py`: the Basic/Advanced session presentation boundary. Both views edit the same scheduler state; Basic exposes the preset, size, Topic/Item mix, Document/Other mix, summary, and preview.
- `backend/session_selection.py`, `backend/scheduler.py`, `backend/topic_scheduler.py`: session candidate filtering, tag-aware selection, refill preview behavior, and reader-card scheduler integration.
- `frontend/command_palette.py`, `frontend/onboarding_dialog.py`, `frontend/activity_center.py`, `backend/activity_log.py`: command discovery, versioned first-run guidance, and bounded user-visible background-task state.
- `frontend/settings_dialog.py`, `config.json`, `__init__.py`: config-backed settings tabs, persisted defaults, and save/load wiring for extraction, review, topics, writing, shortcuts, and advanced tools.
- `backend/note_metadata.py`: shared Incremento provenance fields and helpers. New note-creation paths should use this instead of appending source/parent text into content fields.
- `frontend/add_card_dock.py`, `backend/reviewer_extract.py`, `frontend/extract_batch_dialog.py`: transfer-to-note flows, topic/item tag toggles, batch Q/A extraction, and reviewer-side extract plumbing.
- `backend/extraction_drafts.py`, `frontend/add_card_dock.py`: bounded atomic per-profile extraction-draft autosave plus explicit restore/discard UI.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BaskovicP/Incremento](https://github.com/BaskovicP/Incremento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
