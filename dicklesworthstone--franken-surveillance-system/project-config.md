---
trigger: always_on
description: handles; recommendations require a decomposed nondominated affordance frontier.
---

# AGENTS.md

Read this entire file and `COMPREHENSIVE_PLAN_FOR_FRANKEN_SURVEILLANCE_SYSTEM.md` before editing.
The repository is designed for autonomous coding agents, but it does not permit an agent to infer
missing authority, lower a gate, or replace a specified architecture with a quick substitute.

## Prime directive

Build a deterministic, evidence-native semantic control plane for owner-authorized physical
sensors. Preserve uncertainty and provenance. Keep cognition derived. Make effects explicit,
idempotent, capability-scoped, and later verifiable. Make the whole system legible through the one
canonical linked abstraction tower: runtime authority/custody → source evidence → world facts and
coverage → derived beliefs → SituationCapsule → investigation/hypotheses → affordance frontier →
plan/effect/obligation → outcome/episode → learning/memory → workspace/handoff. Mission and
ObjectiveContract, semantic protocol, privacy/capability projection, and full cost are cross-cutting
coordinates, not rival layers.

## Truth hierarchy

1. Machine-readable registries and versioned schemas.
2. Normative comprehensive plan and ADRs.
3. Tests and retained proof bundles.
4. Implementation.
5. Explanatory documentation.

When these disagree, do not choose the most convenient one. Record the drift, identify the owning
contract, and repair the set coherently.

## Non-negotiable architecture

- Rust 2024 on the pinned nightly toolchain.
- Asupersync only for asynchronous orchestration; no Tokio adapters hidden behind features.
- `#![forbid(unsafe_code)]` in every FSS workspace crate, target, example, test, and build helper; there is no local exception path.
- Production media, model, graph, storage, and protocol semantics are first-party pure Rust. Foreign frameworks/applications are laboratory or migration oracles only.
- `Cx` or an equivalent explicit authority reaches every I/O, time, sleep, lock, network,
  secret, effect, and sealed laboratory-oracle process boundary.
- Region ownership; cancellation is request→drain→finalize; no orphan work.
- Authority, cognition, and effect planes are type-distinct.
- Canonical history is one ordered `EvidenceDeltaBatch` universe. Derived state is anchor-pinned and rebuildable.
- Negative reads require `CoverageWitness`; semantic plans require read/write witnesses.
- ATP moves immutable object graphs only and never carries effect authority.
- Root-last publication distinguishes staged, visible, durable, replicated, protected, and retrievable states.
- Graph algorithms use registered projections, CGSE tie-breaks, and complexity/output witnesses.
- Immutable model/device/config generations.
- Stable IDs are never renumbered; superseded entries remain tombstoned.
- AgentSession, AgentWorkspace, ContractBasis, universal request/response envelopes,
  SituationCapsule, SituationFrame, WorldEnvelope, ContextPack, investigation,
  affordance, plan, episode, and handoff semantics come from the agent machine registries; no
  transport invents a parallel vocabulary.
- Knowledge state, provenance class, hypothesis disposition, access transform, and effect outcome
  remain orthogonal typed fields.
- No mission-critical fact, assumption, obligation, lease, indeterminate effect, or next step may
  exist only in conversational context.
- Compact agent outputs require a semantic compression receipt and priced evidence-hydration
  handles; recommendations require a decomposed nondominated affordance frontier.
- Protected high-loss possible worlds cannot disappear through ranking, model reranking,
  compression, transfer, view changes, or handoff. Every action is classified as robust,
  conditional, information-gathering, wait/watch, blocked, or unavailable against the exact
  `WorldEnvelope`; plans bind its digest and name supported and unsafe worlds.

## Workflow

1. **Restore the mission:** inspect the latest `AgentSessionCapsule`/handoff, current authority,
   active obligations, work claims, invalidations, and negative evidence. Never rely on remembered
   chat state.
2. **Orient from one anchor:** request or construct the smallest sufficient `SituationCapsule` and
   verify its exact `ContractBasis`, inner `SituationFrame`/`WorldEnvelope`, epistemic map,
   compression receipt, resource pressure, and categorized control envelope.
3. **Establish scope:** stable requirement IDs, mission/objective contract, files, authority changes,
   migrations, budgets, affected planes, and qualification gates.
4. **Preserve alternatives:** for uncertain work, create/revise an `InvestigationCase` with
   competing hypotheses, evidence, contradictions, predicted observations, falsifiers, shared
   failure domains, probe costs, and stop rules.
5. **Implement the smallest coherent vertical contract:** begin with deterministic reference
   behavior, explicit failure states, stable handles, and the agent-facing projection—not a fake
   end-to-end demo or isolated subsystem trick.
6. **Make the next move legible:** expose typed nondominated affordances after capability/privacy/
   safety clamps, with value, cost, risk, reversibility, invalidators, and expected proof. A
   recommendation never grants effect authority.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/franken_surveillance_system](https://github.com/Dicklesworthstone/franken_surveillance_system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
