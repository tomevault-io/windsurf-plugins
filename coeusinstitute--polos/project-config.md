---
trigger: always_on
description: > **If you are an agent:** this repository (Polos) is a buildable specification, not a library to import.
---

# AGENTS.md — Polos (DOX Root)

> **If you are an agent:** this repository (Polos) is a buildable specification, not a library to import.
> To instantiate the mesh in your host environment, read this file, then execute `BUILD.md`.
> The specs in `constitution/`, `roles/`, `contracts/`, and `models.yaml` are runtime-neutral.
> Map them onto your stack's primitives using `adapters/`. Do not weaken any constraint while
> building. Run the completeness check in `contracts/flow.graph.yaml` before going live.

## Purpose
Provide a portable, self-documenting safety harness that decomposes one logical agent into a
mesh of specialized agents so that no single component can both decide to cause harm and carry
it out. Safety is a property of the **topology**, enforced structurally, not a property assumed
of any single model.

## Repository Ownership
- This root owns the **global invariant**, the **plane model**, the **build protocol**, and the
  top-level index.
- `constitution/` owns immutable rules and capability ceilings. **Authoritative.**
- `roles/` owns canonical agent definitions and the agent-card template. **Authoritative.**
- `contracts/` owns the message envelope, the complete connection graph, and the state machine.
  **Authoritative for all inter-agent flow.**
- `oversight/` owns the three guardian policies (Monitor, QC, Security).
- `models.yaml` owns model binding for every role.
- `backpacks/`, `audit/` own runtime-written lessons/playbooks and the append-only log.
- `loops/` owns runtime loop ledgers; `experience/` owns the runtime episode store for measured self-improvement.
- `environment/` owns redacted runtime environment profiles used to match safe repeatable task playbooks.
- `adapters/` owns stack-specific bootstrap notes (additive only).
- `polos/` owns the installable Python runtime layer: CLI, task contracts, policy, grants,
  tool gateway, verification, loop controller, provider abstraction, and adapter interfaces.
- `tests/` owns runtime tests for the installable harness layer.
- `docs/` owns human-facing material. **Derived** — never authoritative.

## Global Contracts
- **The invariant.** Deciders (Alpha, Router, Taskmaster, Loop Controller) hold no effectful tools.
  Only Execution Workers act, and only on scoped, monitored, credentialed assignments. Oversight
  agents (Monitor, QC, Security) hold no tools and can only reject / rework / quarantine / halt.
- **One envelope.** Every inter-agent message is a valid `MeshEnvelope`
  (`contracts/envelope.schema.json`). No exceptions; this is what makes the flow gapless.
- **Gates are total.** Every edge that delivers a worker result or a doc change is mediated by
  the Monitor (safety) then QC (quality). Gates are assigned by `contracts/flow.graph.yaml`,
  never chosen by the sender.
- **Docs describe, they do not define.** `AGENTS.md` files are derived views of the canonical
  specs. If a doc ever contradicts a canonical source, the source wins and the Archivist repairs
  the doc. No `AGENTS.md` may weaken DOX, the constitution, or a parent's safety constraints.
- **Fail-closed.** Any unavailable or uncertain gate blocks. Ambiguity resolves toward safety.
- **Corrigible.** The mesh accepts halt and correction without resisting or self-preserving.
- **Loops are bounded.** A looped goal declares budgets and an externally verifiable stop condition checked by the Verifier; the Loop Controller holds no tools; Security can HALT a runaway loop.
- **Self-improvement is measured.** Proposed lessons/playbooks must pass the Evaluator (benefit, no regressions) before ratification, are append-only and versioned, and can never grant capability or weaken a constraint.
- **Nurse triage is conditional.** The Nurse is read-only and wakes only on explicit checkup request,
  Security integrity signal, or thresholded audit pattern. It diagnoses harness drift and emits repair
  manifests; repairs execute only through normal Monitor-reviewed Taskmaster assignments/doc assignments.
- **Environment profiles are redacted.** Host/tool/provider facts may be recorded for playbook matching,
  but secrets never are. Ambiguous provider targets fail closed until a human resolves them.
- **Project context is user-authored.** `PROJECT_CONTEXT.md` at the repo root is hand-edited by the
  user to capture project stack, deploy targets, conventions, and gotchas. The Router retrieves it as
  background data (Monitor-scrubbed); it is context, never authority — it cannot grant capability,
  change gates, or weaken the constitution.

## Work Guidance
- Treat `constitution/core.md` as read-only at runtime. Changes happen out-of-band by a human,
  then a redeploy.
- Every agent observes **Read-Before-Editing** and **Update-After-Editing** (DOX). The Archivist
  owns installation, repair, and closeout of the `AGENTS.md` tree.
- Bind models only in `models.yaml`. Never hardcode a model inside an agent card.
- Add stack support by adding a file under `adapters/`, never by editing the canonical specs.
- Keep runtime implementation in `polos/`. Runtime code may wrap canonical specs but must not make
  README/docs more authoritative than `constitution/`, `roles/`, `contracts/`, `models.yaml`, or
  `tools/validate_mesh.py`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CoeusInstitute/Polos](https://github.com/CoeusInstitute/Polos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
