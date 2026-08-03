---
trigger: always_on
description: This file is for AI agents modifying or operating this repository. The README is the adopter-facing product guide; this file is the codebase working map and invariants checklist.
---

# CLAUDE.md - Contributor Agent Guide

This file is for AI agents modifying or operating this repository. The README is the adopter-facing product guide; this file is the codebase working map and invariants checklist.

Review agents auditing claims should read in this order: `AUDIT.md` -> `docs/CAPABILITIES.md` -> `docs/CONFIGURATION.md`. Treat those documents as claims to verify against source, not as proof.

## Product Shape

`claude-code-fleet-orchestrator` is a local-first, single-user orchestrator for one operator running multiple Claude Code or CLI agent sessions. It is not a hosted service, not a multi-tenant system, and not a SaaS control plane.

Core state is split by purpose:

- Neo4j stores projects, phases, tasks, dependencies, refs, gates, rules, evidence, and project/session relationships.
- Redis stores notification/liveness surfaces, current-task bindings, peer status, and distributed locks.
- The mutable operator API and dashboard run on `:5002`.
- The separate public dashboard runs through `scripts/orch-public` on `127.0.0.1:5005` by default and exposes only read-only routes.

The default network posture is private. `ORCH_HOST` defaults to `127.0.0.1` for the mutable `:5002` API/dashboard; a routable bind is an explicit trusted single-user LAN opt-in. `scripts/orch-public` does not use `ORCH_HOST`; it hardcodes `127.0.0.1` and exposes only `--port`. `ORCH_AUTH_TOKEN` is optional and gates mutable HTTP methods when set. Read endpoints remain open, so loopback binding is still the primary default security boundary.

## Code Map

- `fleet_orchestrator/orch_schema.py` is the domain layer for Neo4j schema setup, task creation, task status transitions, evidence validation, readiness queries, stop decisions, human-review gates, refs, rules, and project/session state.
- `fleet_orchestrator/tasks_api.py` is the FastAPI mutable API. It wires startup schema initialization, optional mutable-method auth, task/project/session routes, stop-decision routes, wake-packet assembly, and dashboard endpoints.
- `fleet_orchestrator/context_assembler.py` builds dynamic wake packets from overall/supervisor/project/phase/task refs, memory, rules, and snapshots. It wraps inlined untrusted content in a nonce envelope; preserve that boundary.
- `fleet_orchestrator/dispatch.py` claims work, binds Redis current-task state, sends notifications, and records worker outcomes with `record_outcome`.
- `fleet_orchestrator/cli_orch_watch.py` is the Redis event/sweep watcher. It wakes supervisors for stuck, unblocked, or ready work and independently watches notify-daemon liveness plus stuck-inbox delivery, alerting out-of-band when notification delivery is at risk.
- `fleet_orchestrator/config.py` defines the environment contract. Required stores include `ORCH_REDIS_HOST`, `ORCH_REDIS_PORT`, `ORCH_NEO4J_URI`, and `ORCH_NEO4J_DB`; use the config helpers instead of ad hoc environment reads for orchestrator settings.
- `fleet_orchestrator/public_readonly.py` is the scrubbed dashboard API: GET-only routes, explicit public session/project filters, and text redaction for local home/user paths before rendering public project/session/task fields. Keep it read-only.
- `scripts/orch` is the operator lifecycle CLI for serving, doctoring, enabling, disabling, and uninstalling the local service.
- `scripts/taey-plan`, `scripts/taey-task`, `scripts/taey-question`, and `scripts/taey-dispatch` are the agent-facing CLIs.
- `ui/` contains the browser dashboard assets.
- `tests/*_acceptance.py` are the main behavioral gates for orchestration invariants.
- `.github/workflows/ship-gate.yml` and `.github/workflows/r5-audit-gate.yml` are the release and risky-surface gates.

## Working Rules

Before editing a function, class, or method, inspect callers, API consumers, and matching acceptance tests with normal repo tools such as `rg`, `git log`, `git blame`, and the existing tests. Keep changes scoped to the requested behavior and verify against committed code, tests, and CLI help, not memory.

Do not bypass these invariants:

- Terminal task transitions require evidence. Completed tasks need at least one well-formed `commit_sha`, `gate_run_id`, or `production_observation`; failed/interrupted states require explanatory terminal evidence.
- Human-review gate tasks cannot be completed by normal agent task updates. They are completed through the dedicated dashboard review path.
- Ready-work and stop-decision behavior are coupled. The canonical readiness readers are `get_ready_tasks()`, `get_session_next_ready()`, `get_project_ready_tasks()`, and `ready_work()` in `fleet_orchestrator/orch_schema.py`; stop behavior reads readiness through `_raw_stop_decision()` / `get_session_stop_decision()`. API consumers include `tasks_api.py` routes for next-ready, stop-decision, and wake-packet assembly. Dispatch consumers include `_claim_ready_orch_task()` and `bind_current_task()` in `fleet_orchestrator/dispatch.py`.
- Wake-packet refs are optional context, not instructions. Untrusted inlined content must stay nonce-wrapped and separated from trusted instruction text.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [palios-taey/claude-code-fleet-orchestrator](https://github.com/palios-taey/claude-code-fleet-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
