---
trigger: always_on
description: This document defines repository-level engineering conventions that apply to every automated or manual change in this repository. In addition to the general requirements, it **strictly governs changes to the `upload_folder` upload path**: every change that touches this path must perform the specified scenario-based simulation review and include an Upload Simulation Regression Report.
---

# AGENTS.md — modelscope_hub Engineering Conventions

This document defines repository-level engineering conventions that apply to every automated or manual change in this repository. In addition to the general requirements, it **strictly governs changes to the `upload_folder` upload path**: every change that touches this path must perform the specified scenario-based simulation review and include an Upload Simulation Regression Report.

---

## 1. General Engineering Requirements

- This repository uses a src-layout. Always run code and tests against local sources: `PYTHONPATH=src` (a package installed with `pip install` may be outdated).
- Before committing, run the following gates in order; all must pass:
  1. Targeted unit tests: `PYTHONPATH=src pytest -q <relevant tests>`
  2. Full non-remote test suite: `PYTHONPATH=src pytest -q tests/ -k 'not remote' --ignore=tests/integration`
  3. Style checks: `ruff check src/ tests/` and `ruff format --check src/ tests/`
  4. Type checks: `mypy src/modelscope_hub/`
- Never commit secrets. `~/.modelscope/credentials/` is private (it contains a pickled cookie jar and `m_session_id`); never expose its contents in logs or reports.
- When changing the default value of a configuration constant, update the corresponding tests for its default value and registration table, as well as the environment-variable table in `README.md`.

---

## 2. Controlled Scope: `upload_folder` Path

Changing any of the following files or symbols constitutes a change to the upload path and requires the mandatory simulation regression in Section 5:

- `src/modelscope_hub/_upload.py`
  - Batching: `_calculate_adaptive_batch_size`, `_plan_commit_batches`, `_estimate_commit_operation_bytes`, `_plan_result_batches`
  - Routing: `_is_lfs`, `_is_inline_metadata`, `_upload_mode`
  - Main flow and recovery: `upload_folder`, `_retry_failed_commits`, `_retry_failed_files_react`, `_retry_failed_simple`, `_commit_with_retry`
  - Failure classification: `classify_error`, `_ErrorCategory`, `_is_retryable_commit_error`
  - Preflight validation: `_warn_advisory_upload_limits`, `_prepare_upload_folder`
  - State tracking: `UploadTracker` (`begin_attempt` / `mark_*`)
- `src/modelscope_hub/errors.py`: `raise_for_status`, `_COMMIT_RETRYABLE_BUSINESS_CODES`, and HTTP/business-code mappings
- `src/modelscope_hub/_legacy_api.py`: `create_commit`, including the HTTP 200 / `Success:false` check
- `src/modelscope_hub/constants.py`: upload-related constants such as `UPLOAD_*` and `COMMIT_MAX_ACTIONS_PER_REQUEST`

---

## 3. Commit Batching Semantics That Must Be Preserved

Only files that are **pending** (not `COMMITTED`) for the current run are planned into batches. This prevents a resumed upload from turning a small number of remaining files into multiple tiny commits.

Apply the following three constraints; start a new batch as soon as any constraint is reached:

1. **Operation target (primary constraint):** `UPLOAD_COMMIT_BATCH_MAX_OPERATIONS`, default `256`. `_calculate_adaptive_batch_size` must return `min(target, 2000, pending_count)`—for fewer files, it converges to the pending count and by default never exceeds `256`.
2. **Request-body budget (secondary constraint):** `UPLOAD_COMMIT_MAX_INLINE_BYTES`, default `8 MiB`. Count only normal files, using base64 expansion (`×4/3`) plus JSON overhead. For LFS files, count only pointer metadata (about `370 B/op`); **never include blob payload bytes**.
3. **Server hard limit:** `COMMIT_MAX_ACTIONS_PER_REQUEST=2000`. Clamp to this limit in every case; it must never be exceeded.

The following behavior must also be preserved:

- **Tail-batch rebalancing:** when `last_batch × 4 < previous_batch` and both rebalanced halves meet all constraints, rebalance the final two batches (for example, `256/32 → 144/144`). Do not add commits, and preserve deterministic path order.
- **Oversized inline fallback:** if one inline file alone exceeds the budget, place it in its own batch and emit a warning; the upload must not stall.
- **Recovery uses the same planner:** `_retry_failed_commits`, ReAct recovery, and simple retry must all use `_plan_result_batches` (and therefore the same `_plan_commit_batches`).
- **`delete_files` batches independently:** split only at the hard limit of `2000`, not at `256`.
- **`upload_file`:** one file and one commit per call; no batching and no tracker.

### Routing Rules: LFS vs. Inline

- Files matched by `_is_inline_metadata` (such as `README.md`, `.gitattributes`, `.gitignore`, and `config*.json`) are **always inline**.
- Otherwise, `size > UPLOAD_LFS_FORCE_THRESHOLD_BYTES` (default **64 KiB**) routes to LFS.
- Otherwise, files matching `DATASET_LFS_SUFFIX` or `MODEL_LFS_SUFFIX` route to LFS.
- All other files are normal inline files.

---

## 4. Failure and Retry Model That Must Be Preserved

- Failures have three categories, and **all of them must be included in final statistics**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [modelscope/modelscope_hub](https://github.com/modelscope/modelscope_hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
