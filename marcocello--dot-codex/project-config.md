---
trigger: always_on
description: - This file is the navigation contract for Codex in this repo.
---

# AGENTS.md - Marco Dev Operating Kernel

## Role
- This file is the navigation contract for Codex in this repo.
- `AGENTS.md` owns global authority, assurance lanes, completion, safety, and handoff rules.
- Shared harness executables are owned by dot-codex and invoked from any checkout through `"${CODEX_HOME:-$HOME/.codex}/scripts/<tool>"`. Do not create, copy, or wrap these tools in target repositories.
- Non-coding or personal operating work routes first through `docs/secondbrain.md` and matching `second-brain-*` skills.
- Skill descriptions own the shortest unique routing trigger or exclusion. Skill bodies own task-specific procedure, examples, stack choices, decision rules, and domain judgment.
- Scripts own command arguments, outputs, exit semantics, and mechanical guarantees.
- Harness docs own durable rationale, threat models, and detailed proof, autonomy, safety, and handoff design.
- Repo docs own app context when present: `docs/APP.md`, `docs/ARCHITECTURE.md`, `docs/CONVENTIONS.md`, `docs/TESTING.md`.
- State each instruction once at its owner. Cross-reference the owner instead of copying defaults or full workflows.

## Work Kernel
- Work on one issue, one feature, and one `FEATURE_DIR` at a time. One accountable parent owns decisions, implementation, proof, evaluation, queue state, and completion.
- Select one assurance lane before editing and keep downstream skills aligned with it.
- `lightweight`: isolated, low-risk edit or bug fix with no durable behavior contract; use the smallest regression or narrow check. No `FEATURE_DIR`, captured proof, evaluator, or queue mutation required.
- `tracked`: feature work, behavior-contract work, or repairs involving queues, safety, data, migrations, external services, multiple modules, or repeated failures; use the normal feature lifecycle.
- `autonomous`: explicit keep-going, queue, or repeated-repair work; use the tracked lifecycle plus persistent recovery and queue state.
- `FEATURE_DIR/FEATURE.md`: behavior contract.
- `FEATURE_DIR/PROOF.md`: realistic proof contract.
- For non-trivial work, use two decision passes before substantial implementation: feature challenge/decision summary, then proof challenge/decision summary. Each pass may contain zero user questions. Ask only when an unresolved user-owned choice has no safe default and its answer can materially change behavior, scope, safety, cost, data, permissions, or external effects. After the user answers, proceed without asking them to approve the written contract. When repository context, the request, or safe defaults resolve the choices, state the decisions and proceed directly.
- Route scope from the requested deliverable, not isolated verbs. A request for feature, proof, specification, or planning artifacts is contract-authoring work and ends after decision-ready `FEATURE.md`, `PROOF.md`, and executable `proof/run.sh` artifacts are written. Answers to discovery or proof questions do not authorize implementation. Contract authoring must not invoke `coding-feature-execute`; implementing the described product behavior requires a separate explicit request.
- Do not claim completion from plausibility, source shape, assistant claims, tool-call success, a gate, or an evaluator without realistic executable proof.
- For issue work, first check whether the defect clearly belongs to `docs/features/*/FEATURE.md`.
- Exactly one match: use that `FEATURE_DIR`; add a focused regression when current proof misses the defect.
- No clear match: use the smallest local regression unless expected behavior needs durable product definition. Do not create a feature package merely because the harness exists.
- New or materially changed product behavior without a clear owner: create `docs/features/<request-slug>/FEATURE.md`, `PROOF.md`, and executable proof.
- Semantic behavior must be fixed at the owning invariant, not through open-ended keyword, phrase, or language lists. Hardcoded lists are valid only for closed vocabularies from protocols, enums, provider contracts, product taxonomies, or explicit specs.
- Ambiguity checkpoint: before editing, state the intended behavior, rejected material alternative, and consequence when multiple strategies, auth/secrets/deployment/runtime/data, exact paths/sources, or a user correction could change the result. Ask focused questions when unresolved.
- Correction checkpoint: after a user correction, restate the accepted behavior and rejected previous direction before editing again.
- Promote lightweight work to tracked or autonomous when it touches behavior contracts, queues, safety, data, migrations, external services, multiple modules, or repeated failures.

## Completion Kernel
- Lightweight work is complete after its focused regression or narrow check passes; add broader checks only when the touched surface justifies them.
- Tracked and autonomous work require a passing realistic proof and a fresh read-only `coding-feature-evaluator` `PASS` for the current implementation and proof.
- Every tracked and autonomous feature invokes the evaluator after proof passes. Lightweight work does not invoke the evaluator.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcocello/dot-codex](https://github.com/marcocello/dot-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
