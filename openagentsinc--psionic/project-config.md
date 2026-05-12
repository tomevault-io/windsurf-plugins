---
trigger: always_on
description: Write directly. Prefer blunt declarative sentences over rhetorical scene-setting.
---

# Psionic Agent Contract

## Writing style

Write directly. Prefer blunt declarative sentences over rhetorical scene-setting.

Do not use contrastive metadiscourse or rhetorical throat-clearing such as:

- "the interesting question is not X, it is Y"
- "the real question is..."
- "the point is not..., the point is..."
- "X lives in a different neighborhood"
- "X sits in a different orbit / layer / space"

Do not use vague conceptual metaphors when a direct architectural statement will
do.

Bad:

- "The HyperAgents paper lives in a different neighborhood."
- "The interesting question is not whether this is impressive in the abstract."

Good:

- "HyperAgents improves worker, validator, and routing quality inside OpenAgents."
- "This matters because it lowers cost per accepted outcome."

Default to:

- direct claims
- explicit subsystem names
- concrete causal language
- plain statements of what is true, what changes, and why it matters

## Scope

- This repository is the standalone `psionic` workspace extracted from
  `openagents`.
- Primary scope is `crates/psionic-*`, `docs/`, `fixtures/`, and the small set
  of repo-local `scripts/`.
- Psionic owns the machine-facing execution substrate: tensor and compiler
  contracts, runtime and backend truth, serving interfaces, cluster and sandbox
  execution, adapters, data, eval, research, and the early training substrate.
- Keep app UX, wallet or payout flows, market orchestration, and kernel or
  settlement authority out of this repo unless the user explicitly asks for
  cross-repo work.

## Canonical Docs

- `README.md` is the Psionic entrypoint and map.
- `docs/ARCHITECTURE.md` is the canonical Psionic-wide system spec.
- `docs/FRAMEWORK_CORE_ACCEPTANCE_MATRIX.md` is the canonical framework-core
  completion bar.
- `docs/INFERENCE_ENGINE.md` is the canonical inference-engine completion doc.
- `docs/TRAIN_SYSTEM.md` is the canonical training subsystem spec.
- Domain-specific docs in `docs/` define the contract for their subsystem.
- `docs/audits/` explains rationale and follow-on direction, but audits are not
  the canonical current-state spec.
- For Tassadar paper-backed roadmap or issue work, the local paper corpus lives
  in `~/code/alpha/tassadar/tassadar-research/papers/` and
  `~/code/alpha/tassadar/can-llms-be-computers/papers/`.
- The corresponding local reading notes live in
  `~/code/alpha/tassadar/tassadar-research/notes.md` and
  `~/code/alpha/tassadar/can-llms-be-computers/notes.md`.

## Workspace Map

- Core execution path lives in crates such as `psionic-core`, `psionic-array`,
  `psionic-ir`, `psionic-compiler`, `psionic-runtime`, and the backend crates.
- Distributed and execution plumbing lives in crates such as
  `psionic-cluster`, `psionic-collectives`, `psionic-distributed`,
  `psionic-datastream`, `psionic-sandbox`, and `psionic-net`.
- Serving and provider-facing compute surfaces live in crates such as
  `psionic-serve`, `psionic-provider`, `psionic-router`, and
  `psionic-catalog`.
- Data, model, training, eval, environment, and research lanes live in crates
  such as `psionic-data`, `psionic-models`, `psionic-train`,
  `psionic-environments`, `psionic-eval`, `psionic-research`,
  `psionic-adapters`, and `psionic-apple-fm`.
- `fixtures/` contains committed evidence, run bundles, and compatibility
  artifacts. Treat them as versioned substrate truth, not disposable samples.

## Execution Rules

- Read the relevant canonical doc before editing a subsystem.
- Keep machine-legible truth explicit: manifests, receipts, proofs, capability
  reports, refusal reasons, runtime identity, artifact identity, and lineage
  should remain accurate and deterministic.
- Preserve replay-safe and deterministic behavior. Do not hide fallbacks,
  bounded support, or refusal posture behind optimistic defaults.
- Prefer extending existing crate boundaries over adding cross-cutting
  shortcuts or hidden control planes.
- If behavior or architecture changes, update the relevant doc in `docs/` in
  the same change.
- Use the repo status vocabulary consistently:
  `implemented`, `implemented_early`, `partial`,
  `partial_outside_psionic`, and `planned`.
- If a host, credential, external machine, or network path is blocked, do not
  sit idle waiting for the user by default.
- Route around the obstacle when possible: keep working on accessible devices,
  reduce the active scope to the honest reachable set, and update issues or
  docs so the recorded plan matches reality.
- Only stop to ask the user when the blocked dependency is mandatory for the
  next honest step and there is no meaningful non-blocked work left.

## Worktree Hygiene

- Start every task by checking `git status --short --branch`.
- If the checkout already contains unrelated changes, do not run broad
  formatters, artifact-generating examples, or repo-wide update commands in
  that checkout.
- Commits and pushes land on `main` unless the user explicitly instructs
  otherwise.
- Prefer working directly in the main checkout and cleaning up task-owned dirt
  there rather than creating extra worktrees or issue branches.
- Never clean a dirty tree by resetting, discarding, or reverting changes you
  did not make.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenAgentsInc/psionic](https://github.com/OpenAgentsInc/psionic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
