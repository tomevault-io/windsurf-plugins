---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Screencap is a macOS CLI for screen recording. The recording engine lives at `src/screencap/engine/` as an internal sub-package. Python >= 3.10, macOS only.

A native SwiftUI app shell lives at `macos/` and wraps the bundled CLI; see "macOS SwiftUI app shell" below for details.

A Chrome extension lives at `extension/` (TypeScript, Manifest V3) — the browser tier, which records browser work without a macOS install. It is a self-contained npm sub-project with its own toolchain and shares no build with the Python layer; the Python conventions below do not govern it. Its viewer half lives in the sibling `screencap-website` repo. See `docs/plans/2026-07-29-001-feat-chrome-extension-browser-tier-plan.md`.

### Privacy subsystem packages (SCR-33)

The privacy code is split into three sibling packages forming a one-way dependency DAG (`privacy` is the leaf; both halves depend on it; neither depends on the other):

- **`src/screencap/privacy/`** — the **shared privacy-model core** (leaf): the action vocabulary (`actions.py` — `PrivacyAction`, action-sets), the context×mode policy matrix (`policy.py`), the context classifier + bundle/domain maps (`classify.py`), audit records (`reasons.py` — `AuditEntry`/`ReasonCode`), the domain index loader + its bundled `data/ut1` blocklists (`domain_loader.py` + `data/`), and the pixel/region masking primitives shared by both halves (`mask_primitives.py`). Its `__init__` deliberately re-exports nothing, so importing a small shared symbol never pulls in heavier modules.
- **`src/screencap/enforcement/`** — **capture-time enforcement** (runs on every window event during recording): `recorder_enforcement.py` (`RecorderPrivacyFilter`), `window_filter.py` (the cloud/local window-title filters; the call-graph guard `tests/test_privacy_filter_call_graph.py` pins `build_privacy_filter`'s home here), `persistence.py`, `disable_log.py`, and `scrub_worker.py` (a capture-time row-deletion sidecar — *not* a text scrubber). Carries no NLP/ML import surface.
- **`src/screencap/redaction/`** — **post-hoc detection + redaction** (runs during scrubbing): the standalone string-detection/anonymization engine (`engine.py` + `pii`/`regex`/`secrets`/`resolver`/`filters`/`entity_mapping` backends), Apple Vision OCR (`ocr.py`), policy-driven image-mask orchestration (`masking.py`), and the scrub-time DB-geometry readers (`geometry.py`). Invoked through the narrow `create_default_pipeline` / `Anonymizer` / `normalize_text` seam.

`tests/test_package_boundary_call_graph.py` and `tests/redaction/test_import_lightness.py` are the durable guards for this DAG and for the shared core staying light.

### Daemon architecture (Phase 2)

The recording engine is supervised by a background daemon. CLI live-state commands (`screencap start` / `stop` / `status`) are thin HTTP clients of the daemon's `/v0/*` API over a UNIX socket at `~/.screencap/run/api.sock`. The daemon itself runs via `screencap serve` and is normally managed by a LaunchAgent installed by `screencap setup`.

When a CLI live-state command runs on a machine with no LaunchAgent installed (the headless / F3 install case), the CLI **auto-spawns** the daemon in the background via `posix_spawn` with `--idle-shutdown=600`. The auto-spawned daemon exits cleanly after 10 minutes of no requests, no event subscribers, and no active recording — preventing cron-driven `screencap status` from leaving permanent background processes. LaunchAgent-managed daemons omit the flag and run all day.

Auto-spawn diagnostic log: `~/.screencap/run/auto-serve.log` (mode 0o600).

### Unified recording processing pipeline

Recordings flow through one **disk-first pipeline** rather than a fork at recording start. Capture writes rich chunks to disk as the source of truth; destination-agnostic stages run once per chunk; a single terminal stage converges each recording toward its destination.

- **On-disk per-chunk ledger** — `src/screencap/pipeline_state.py` (`PipelineLedger`) persists per-chunk lifecycle state in a `pipeline_chunk_state` table inside the local-only `recording.db`. It is the on-disk replacement for the old in-memory `chunk_processor._chunk_results`, re-establishing the five data-loss prevention rules on disk (closed-set seeding at rotation, tri-state upload state where `SKIPPED` ≠ `UPLOADED` ≠ `FAILED`, never-delete-without-fresh-remote-confirm, frozen `chunks_expected`, crash-safe `EVICT_PENDING` ordering). The ledger is written cross-process with `busy_timeout=10000` + `BEGIN IMMEDIATE` per transition.
- **Destination-agnostic stages** — `src/screencap/pipeline_stages.py` (`PipelineStageRunner`) runs transcribe → export events → manifest once per chunk, idempotently (ledger `STAGED` + on-disk artifact existence), with no local-vs-cloud knowledge.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [proteus-computer-use/screencap](https://github.com/proteus-computer-use/screencap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
