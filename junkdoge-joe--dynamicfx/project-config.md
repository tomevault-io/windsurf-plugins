---
trigger: always_on
description: This file is the mandatory entry point for every new development session. The repository, not chat history or model memory, is the project record.
---

# DynamicFX repository instructions

This file is the mandatory entry point for every new development session. The repository, not chat history or model memory, is the project record.

## Required reading order

Before proposing or changing implementation:

1. Read this file completely.
2. Read [docs/IMPLEMENTATION_STATUS.md](docs/IMPLEMENTATION_STATUS.md) for the only authoritative current state and exact next action.
3. Read [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the approved target design.
4. Read the relevant Accepted ADRs listed in [docs/adr/README.md](docs/adr/README.md).
5. Read the audit for the current milestone from [docs/audits/README.md](docs/audits/README.md).
6. Read [docs/TEST_MATRIX.md](docs/TEST_MATRIX.md); never infer test status from prose elsewhere.
7. Read [docs/ROADMAP.md](docs/ROADMAP.md) for milestone scope and exit criteria.
8. Inspect `git status`, recent commits, and the diff before acting.

If these sources disagree, stop implementation and reconcile them in this order:

1. Accepted ADRs for decisions;
2. `ARCHITECTURE.md` for target design;
3. `IMPLEMENTATION_STATUS.md` for current reality;
4. `TEST_MATRIX.md` for verification truth;
5. `ROADMAP.md` for planned sequence.

`docs/CONCEPT.md` and `docs/SHADER.md` are prototype snapshots, not target contracts.

## Approved non-negotiable decisions

Do not reopen these decisions during ordinary implementation. A change requires a new ADR that explicitly supersedes the relevant Accepted ADR.

- DynamicFX is an open shader runtime controlled through ordinary After Effects properties.
- The brand is `DynamicFX`; the AE effect and match name remain `DynamicFx`.
- The unreleased prototype is rewritten in place. Do not create `DynamicFx2` and do not preserve prototype parameter indexes or wire formats.
- The selected `Language` popup plus committed `Source.expression` are the source authority.
- `Language` is non-time-varying, defaults to GLSL, and selects an extensible `LanguageFrontend` by stable numeric ID.
- Multi-pass `RenderGraph` is core in Phase 1. Single-pass is a one-pass graph, not a separate runtime.
- Shader parameters use a fixed AE pool, stable `ParamId`s, atomic `BindingPlan` publication, and normal keyframed AE streams.
- Render-side code never calls AEGP. UI/render clones use a newly designed `StateToken` plus sequence schema v1.
- Do not retain prototype `SourceChannel`, flattened v1-v3 migration, legacy `SourceData`, or the sidecar.
- Compile transaction/generation is session-local and must never be persisted.
- `PipelineKey` is based on per-pass artifact and pipeline/device state; it is not derived directly from `DefinitionHash`.
- Multi-pass identities remain separate: module, artifact, graph, definition, pipeline, execution plan, and frame resource.
- Image correctness comes before performance: 8/16/32-bpc and alpha/color behavior precede SmartRender and MFR.
- Target hosts are Windows After Effects 2023, 2024, 2025, and 2026. Apple Silicon macOS follows only after Windows is stable.
- Core rendering does not depend on an editor, WebSocket, cloud service, account, store, licensing layer, or telemetry.
- A local effect package may be added later; an editor is optional and deferred.
- This public repository is the single project record — code and governance corpus together (ADR-0036). There is no private half; the archived `dynamicfx-dev` is history, never a write target.

Full consequences and rationale are in [docs/adr/README.md](docs/adr/README.md).

## Development sequence

Develop by vertical, visible milestones rather than broad refactors with no AE output:

- M0 Architecture Contract
- M1 New-architecture First Frame
- M2 Keyframed Parameters
- M3 Persistence and Render Clone
- M4 Multi-pass Graph
- M5 16/32-bpc Image Quality
- M6 Temporal Feedback
- M7 Performance, SmartRender, and MFR

The exact current milestone and next action live only in [docs/IMPLEMENTATION_STATUS.md](docs/IMPLEMENTATION_STATUS.md). Exit criteria live only in [docs/ROADMAP.md](docs/ROADMAP.md).

Do not silently pull work from a later milestone into the current one unless it is required to preserve an approved core boundary. If that changes scope or order, update the roadmap and record the reason before implementation.

## Required session workflow

### At session start

1. Follow the required reading order.
2. Confirm the working tree and current branch.
3. Restate the current milestone and exact next action from `IMPLEMENTATION_STATUS.md`.
4. Verify that the planned change fits the current milestone and Accepted ADRs.
5. Mark the active work in `IMPLEMENTATION_STATUS.md` before broad implementation begins.

### During implementation

- Keep host-specific AE code outside the definition and graph domain layers.
- Do not add a new persistent field, parameter index, Language ID, graph grammar rule, Shader ABI rule, hash domain, or history semantic without the appropriate ADR.
- Write tests with the contract, not after the implementation is considered finished.
- On discovering an out-of-scope defect, record it as a blocker/follow-up; do not expand the current milestone silently.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JUNKDOGE-JOE/dynamicfx](https://github.com/JUNKDOGE-JOE/dynamicfx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
