---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hermes Memory-OS is a file-first memory and governance runtime that plugs into Hermes agents as a `memory_os` provider. It manages profile-local canonical memory, owner review/approval workflows, governed automatic lanes, and monitor evidence — without owning conversation, transport, or scheduling (those remain Hermes' domain).

## Build & Test Commands

```bash
# Install dev dependencies (Python 3.11+ required)
python -m pip install -e ".[dev]"

# Run full test suite
python -m pytest -q

# Run a single test file
python -m pytest -q tests/plugins/memory/test_memory_os_owner_actions.py

# Run a single test by keyword
python -m pytest -q -k "test_approve_candidate"

# Static checks (run before committing)
python scripts/memory_os_import_cycle_check.py --repo-root .
python scripts/memory_os_write_surface_check.py
python scripts/memory_os_static_hygiene_check.py
python scripts/memory_os_public_checkout_probe.py --source working-tree --strict
git diff --check

# Remote probes (require SSH access to configured hosts)
python scripts/memory_os_cron_adapter_probe.py --host hermes-media --output json
python scripts/memory_os_boundary_runtime_probe.py --host hermes-media --output json
python scripts/memory_os_3_200_monitor.py --host hermes-media --monitor-profile live --output summary
python scripts/memory_os_3_200_monitor.py --host hermes-feiniu --monitor-profile clean-host --output summary
```

Tests live under `tests/plugins/memory/` (core/provider) and `tests/scripts/` (scripts/integration). Test files mirror source modules 1:1 by naming convention.

## Architecture

Hermes is the host agent. Memory-OS is a governed plugin with these layers:

### Core Provider (`plugins/memory/memory_os/`)

- **`__init__.py`** — `MemoryOSProvider`: the main entry point. Hermes calls `initialize` → `prefetch` (read context) → `sync_turn` (write summary-only events to the event queue). Owner-review commands are excluded from turn sync to avoid contaminating memory.
- **`runtime.py`** — `MemoryOSRuntime.heartbeat()`: drives the main processing loop — SessionMirror auto-apply **first** (its events are read by the same cycle — a deliberate latency win, not isolation) → event read + stats cache → candidate generation → working memory decay/prune → state write → index sync. Wrapped in an ExecutionGate `runtime_heartbeat_core` envelope.
- **`cognitive_loop.py`** — `CognitiveLoopRunner.run_once()`: orchestrates portable modules, signal collection, memory projection, and left-brain advisor runs. Four steps open explicit ExecutionGate envelopes (ground_truth_miner, memory_projection, left_brain_advisor, spontaneous_expression); the remaining steps rely on module-internal governance surfaces (StructuralWriteGate etc.).
- **`owner_actions.py`** — `OwnerActionProcessor`: the state-changing entry point. Processes `approve`, `reject`, `feedback`, `allow`, and bounded `apply` actions via stable `oa_` tokens generated in owner digests. High-risk transitions (crystallized writes, revoke/demote/delete, identity writes, external sends) are permanently owner-gated.
- **`execution_gate.py`** — per-execution machine permit system. Creates permit envelopes (`permit_id`, `lane_id`, `risk_class`, `scope`, `boundary`), validates scope, records completion postcheck. Applied to heartbeat, cognitive loop, and cron helpers.
- **`structural_write_gate.py`** — `append_governed_jsonl()`: gates all automatic JSONL writes through ExecutionGate permits. The `write_surface_check.py` script enforces `unclassified_count=0` across the codebase.

**Important naming distinction**: `execution_gate.py` is the *runtime* ExecutionGate (machine permit for automatic work). `plugins/modules/governance/ops_gate.py` is the *proposal* OpsGate (report-only follow-up review of proposals). These are different systems — don't confuse them.

### Memory & Retrieval Layer

- **`store.py`** — file-first primitives: JSONL append/read, atomic JSON writes, quarantine handling.
- **`index.py`** — SQLite index for search and prefetch. Rebuildable from canonical files — never treat the index as the source of truth.
- **`roots.py`** — resolves profile/platform paths (`HERMES_HOME`, `memory-os/` subdirectories).
- **`prefetch.py`** — assembles context for Hermes from task anchor, runtime facts, context router hints, low-clue recall, memory sources, and substrate provider facts.
- **`context_router.py`** — provides context route hints for prefetch decisions.
- **`session_mirror.py`** — bounded session import lane. Owner-graduated (via `approve_session_mirror_apply`), then auto-applied by heartbeat with per-run session caps and platform allowlists. Append-only, no crystallized/policy/identity writes.
- **`crystallized.py`** — owner-approved canonical memory. Candidates require explicit owner approval before becoming crystallized. Supports candidate triage (promote/demote/fleeting/discard) with auto-demote TTL.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [btnalit/Hermes-Memory-OS](https://github.com/btnalit/Hermes-Memory-OS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
