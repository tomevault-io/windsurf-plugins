---
trigger: always_on
description: intelliflow-core is the shared SDK for IntelliFlow OS. It provides governance UI
---

# CLAUDE.md — intelliflow-core

## Package Overview

intelliflow-core is the shared SDK for IntelliFlow OS. It provides governance UI
components, Pydantic contracts, and helper utilities used by SupportFlow and CareFlow.

---

## intelliflow-core v2 — Governed Agentic Runtime

### Pattern: Strangler Fig

v2 is implemented as a sub-package (`intelliflow_core/v2/`) that coexists with v1.
The Strangler Fig pattern means:
- **v1** (`intelliflow_core.contracts`, `intelliflow_core.helpers`, `intelliflow_core.governance_ui`)
  remains 100% unchanged and fully functional.
- **v2** (`intelliflow_core.v2.*`) is opt-in. No v1 consumer is affected.
- Over time, new features are built in v2. v1 is eventually deprecated, not deleted.

### v2 Package Path

    intelliflow_core/v2/
    intelliflow_core/v2/__init__.py              — Package init, re-exports
    intelliflow_core/v2/runtime/__init__.py      — Runtime sub-package
    intelliflow_core/v2/runtime/workflow.py      — LangGraph wrapper (Workflow class)
    intelliflow_core/v2/runtime/state.py         — Shared state schema (IntelliFlowState)
    intelliflow_core/v2/runtime/interceptors.py  — Interceptor node base class
    intelliflow_core/v2/runtime/exceptions.py    — v2-specific exceptions
    intelliflow_core/v2/runtime/contracts.py     — GovernanceRule + WorkflowResult
    intelliflow_core/v2/runtime/kill_switch.py   — KillSwitchGuard(InterceptorNode)
    intelliflow_core/v2/runtime/tool_registry.py — MCPRegistry + ToolSchema (static catalog, dynamic scoping)
    intelliflow_core/v2/storage/__init__.py      — Storage sub-package (exports DatabaseSessionManager, WORMLogRepository)
    intelliflow_core/v2/storage/db.py            — DatabaseSessionManager (SQLite + WAL)
    intelliflow_core/v2/storage/worm_logger.py   — WORMLogRepository (HMAC-SHA256 hash chain, SQLite triggers)
    intelliflow_core/v2/storage/token_ledger.py  — TokenLedgerRepository (append-only token accounting)
    intelliflow_core/v2/tests/__init__.py
    intelliflow_core/v2/tests/test_workflow.py   — 15 tests
    intelliflow_core/v2/tests/test_kill_switch.py — 8 tests
    intelliflow_core/v2/tests/test_tool_registry.py — 12 tests
    intelliflow_core/v2/tests/test_worm_logger.py — 12 tests
    intelliflow_core/v2/tests/test_token_ledger.py — 13 tests

### v2 Entry Point

    from intelliflow_core.v2 import Workflow, IntelliFlowState, InterceptorNode

### v1 Isolation Guarantee

Importing `intelliflow_core` (v1) MUST NOT import LangGraph or any v2 code.
v2 is only loaded when explicitly imported via `intelliflow_core.v2`.

### v2 Dependencies (NOT in pyproject.toml)

v2 requires additional dependencies not listed in pyproject.toml (to avoid
breaking v1 consumers who don't need LangGraph):

    langgraph>=0.2.0     # LangGraph StateGraph API
    Python >=3.10        # Required by langgraph

Install separately:

    pip install "langgraph>=0.2.0"

### Frozen State Pattern

IntelliFlowState uses `ConfigDict(frozen=True)`. To update state:

    new_state = state.model_copy(update={"step_name": "next"})

### Interceptor Architecture

InterceptorNode is the ABC for governance hooks between workflow edges.
- **Step 1 (done):** Define the interface and interceptor slot registration
- **Step 2 (done):** Kill-switch guard implements InterceptorNode
- **Step 3 (done):** MCP Tool Registry — static catalog + dynamic scoping
- **Step 4 (done):** WORM Audit Log — HMAC-SHA256 hash chain, fail-closed, trace_id
- **Step 5 (done):** Token FinOps Tracker — append-only cost ledger, immutable receipt pattern

### Kill-Switch Guard (Step 2)

**GovernanceRule** — description field is required (not optional). Every rule must
be self-documenting for compliance audit trails.

**Fail-closed** — if a rule's logic() raises any exception, it is treated as a
failure. Raw exceptions never escape the kill-switch. This is non-negotiable
for regulated deployments.

**Collect-all-failures** — every rule is evaluated before raising KillSwitchTriggered.
No short-circuit. The full failure set is available for audit logging.

**WorkflowResult** — Workflow.run() returns a structured payload, never a raw
exception to the caller. success=False + kill_switch_triggered=True + failed_rules
list provides everything the caller needs for graceful handling.

**WORM hook** — KillSwitchTriggered carries full GovernanceRule objects (not just
rule_id strings). Step 4 WORM logger reads these directly for immutable audit records.

### MCP Tool Registry (Step 3)

**ToolSchema** — Pydantic BaseModel (not dataclass) with `arbitrary_types_allowed=True`.
Fields: name, description (required), input_schema (JSON Schema dict), callable.
Pydantic validates at registration time, not at retrieval time.

**Static registration** — tools are registered via explicit `register()` calls at
import time. After `lock()`, the registry is immutable. Any post-lock `register()`
raises `RegistryLockedError`. Runtime tool registration is categorically rejected.

**Dynamic scoping** — `get_tools(allowed_names)` returns only the tools whose names
are in allowed_names. Each LangGraph node receives its authorized subset only.
Enforces Least-Privilege Access at the registry level, not by convention.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kmufti7/intelliflow-core](https://github.com/kmufti7/intelliflow-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
