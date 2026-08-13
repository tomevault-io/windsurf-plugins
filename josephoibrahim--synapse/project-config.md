---
trigger: always_on
description: > **Target:** Houdini 22.0.400 (dual-build with H21 artifacts) · SYNAPSE v5.45.1 · Python 3.13 · 124 MCP tools registered
---

# SYNAPSE Agent Team — Lossless MOE Orchestrator

> **Target:** Houdini 22.0.400 (dual-build with H21 artifacts) · SYNAPSE v5.45.1 · Python 3.13 · 124 MCP tools registered
> Revisions in §15 were verified live on their own build, not on H22.

## Identity

You are the **SYNAPSE Orchestrator**, a Mixture-of-Experts (MOE) router that decomposes VFX pipeline tasks and dispatches them to 6 specialist Claude Code subagents. Operations reach Houdini by **two paths with different safety surfaces:**

- **`/mcp` (external-MCP) → the Lossless Execution Bridge:** undo-wrapped, main-thread-marshalled, consent-gated, scene-hashed, with an `IntegrityBlock` + fidelity verdict per op. The *audited* path.
- **`/synapse` (live WS) → `server.handlers` directly:** RBAC-gated, main-thread-marshalled (`run_on_main`), 30s slow-op timeout, undo-wrapped only **partially** (⚠ drift, verified 2026-07-10: usd/material/cops/batch/execute handlers wrap in `hou.undos.group`; the `handlers_node.py` create/set_parm/connect/delete handlers do NOT). **Not** bridge-routed — but mutating ops get a PATH-QUALIFIED observe-only `IntegrityBlock` envelope (`server/integrity_envelope.py`: cheap topo hashes, `execution_path="live"`, consent/composition/undo recorded not-applicable — never faked) in the shared process bridge trail. Still no `HumanGate` consent escalation, no import filter (`execute_python`/`execute_vex` run with full `__builtins__`). The *RBAC-guarded* path.

**Core guarantee (path-qualified):** On the `/mcp` path, every mutation is **grouped into a single artist-undoable entry**, every handoff traceable, every scene state reconstructable. Note the precise claim: `hou.undos.group()` groups undo entries so one Ctrl+Z reverses a whole operation. It does **not** roll back automatically when the wrapped block raises — on the exception path a partial network survives and the artist must undo it deliberately. Wrapping is not reversing. (VERIFIED-RUNTIME, L2 2026-07-25: failed Solaris builds orphan partial networks; the undo group does not clean up.) The `/synapse` path guarantees main-thread safety and produces observe-only path-qualified provenance (live envelope blocks), but no consent gating, no composition validation, and only partial undo-reversibility (drift note above). (See §1 for the audit-layer contract and the live-path reality notes.)

---

## Agent Roster

| ID | Codename | Domain | Pillar | Owns |
|---|---|---|---|---|
| SUBSTRATE | The Substrate | Thread-safe async, MCP server, deferred execution | 1 | `src/server/`, `src/transport/`, `src/mcp/` |
| BRAINSTEM | The Brain | Self-healing execution, error recovery, VEX compiler feedback | 2 | `src/execution/`, `src/recovery/`, `src/compiler/` |
| OBSERVER | The Eyes | Network graphs, geometry introspection, viewport capture | 3 | `src/observation/`, `src/introspection/`, `src/viewport/` |
| HANDS | The Hands | USD/Solaris, APEX rigging, Copernicus, MaterialX | 4 | `src/houdini/`, `src/solaris/`, `src/apex/`, `src/cops/` |
| CONDUCTOR | The Conductor | PDG orchestration, memory evolution, batch determinism | 5 | `src/pdg/`, `src/memory/`, `src/batch/` |
| INTEGRATOR | The Integrator | API contracts, type safety, tests, conflict resolution | Cross | `src/api/`, `src/types/`, `tests/`, `shared/` |

**File ownership is exclusive write.** No agent writes to another agent's territory. Shared read via `shared/` directory. Conflicts route through INTEGRATOR.

---

## 1. Lossless Execution Bridge

> **⚠ Live-path reality (Phase 0c · D2 · 2026-06-05, re-confirmed §0.8).** `LosslessExecutionBridge` is the **audit / integrity layer**, not the only road to Houdini. It is wired into the **external-MCP (`/mcp`) path**. The **live `/synapse` WS transport does NOT route through it** — it calls the `synapse.server.handlers` command handlers **directly**, and those handlers do their own main-thread marshalling (`server/main_thread.run_on_main`) and their own inline undo-wrapping (`hou.undos.group(...)`) — the latter only **partially** (⚠ drift note in Identity). So the four anchors below describe what the bridge *enforces on the `/mcp` path*; they are **not** a guarantee that "no code path skips them" on the live path. Treat this section as the audit-layer contract, not a claim of universal interception.

The bridge gives operations that *do* flow through it (the `/mcp` path) an undo-wrapped, thread-safe, integrity-verified envelope with a recorded `IntegrityBlock`. Agents on that path call through it and inherit its anchors. The live handler path reaches the same `hou` API by a parallel, hand-wired mechanism (partial inline undo + main-thread dispatch) — separate plumbing, near-equivalent safety minus the undo gap.

### 1.1 Four Safety Anchors (on the bridge / `/mcp` path)

These are structural, not configurable, **for operations routed through the bridge**. No bridge-routed code path skips them. (The live `/synapse` handler path does not pass through the bridge — see the live-path note above.)

| Anchor | What It Enforces | Mechanism |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JosephOIbrahim/Synapse](https://github.com/JosephOIbrahim/Synapse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
