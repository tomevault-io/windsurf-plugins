---
trigger: always_on
description: A self-hosted pipeline that turns AI developer workflow traces into RL-ready
---

# AGENTS.md — Sediment

## What Sediment Is

A self-hosted pipeline that turns AI developer workflow traces into RL-ready
training data. It captures inference calls, developer accept/reject decisions,
Edit observations (retention within a Session plus external line deltas), Retry
linkages, git pushes, and CI outcomes as **immutable Facts**; derives
Attributions, edit retention scores, and Reward linkage as **pure,
recomputable functions** over those Facts; and
exports two canonical artifacts — Attributed completions and Rollouts —
projected into DPO/SFT and RLVR training rows (plus the Fact-derived Recovery
pair, ADR 0004's one sanctioned exception). Full picture: `docs/explanation/how-sediment-works.md`.
The architecture is governed by the
ADRs in `docs/adr/` (0001–0005 core, 0006 the open-core boundary, 0007
client-side transcript parsing, 0008 structured inference calls, 0009 canonical
Attribution, 0010 canonical CI outcomes, 0011 training-objective evidence, and
0012 the PostgreSQL-only Fact store, 0013 Git-note observation boundaries, 0014 factual outcomes and training evidence,
0015 lossless representation and bundle v2, 0016 bundle derivation consistency, 0017 bounded sender transport storage, 0018 credential authorities, 0019 repository identity and renames, 0023 indexed call identifiers, and 0024 targeted commit investigations)
— read them before changing anything structural. Current status is `CHANGELOG.md` plus the GitHub milestones.

Python 3.12 for pipeline code. Only `shims/` permits TypeScript (pi requires it).
Harness extensions use their own API; sediment-specific shims live here.
CI scopes Node to `shims/`; nothing else may add a second toolchain.

## The Non-Negotiable Rules

1. **Facts are the only persisted domain state.** ADR 0017 permits an opt-in transport buffer that Derivations never read. ADR 0023 permits exact physical copies of captured call identifiers for indexed lookup; no attachment or policy output belongs there. A Fact is something that happened:
   an inference call, a decision, a push, a CI outcome. Facts are appended, never
   mutated. If you find yourself writing an UPDATE on a Fact table or storing
   the output of a matcher/policy, stop — that belongs in a Derivation.
2. **Derivations are pure functions** of (Facts, policy) and must be
   recomputable over all history. No Derivation may depend on wall-clock
   ingest order or webhook arrival order.
3. **Sessions are the aggregate root.** Developer-side Facts carry a
   `session_id`; the Session row is upserted at the storage seam. Never
   invent placeholder Session ids.
4. **Dedup lives in the database.** `UNIQUE` indexes enforce idempotency —
   do not re-implement dedup scans in application code.
5. **Schema is the source of truth.** All Fact shapes live in
   `packages/core/sediment_core/models.py`. Never define Fact shapes inline
   elsewhere. Derived shapes (`Attribution`, `Rollout`, `Turn`, `CommitRef`,
   `AttributedCompletion`, `RecoverySample`, `Provenance`, every `*Policy`) are frozen
   dataclasses — Pydantic is only for Facts, the notes wire contract,
   settings loaders, and HTTP request envelopes. A field that is a join key,
   a dedup-index component, or the tenancy key gets a validated type
   (`NonEmptyId`, `CommitSha`, `OrgId`, `RepoSlug`, `BranchName`,
   `AwareDatetime`), never a bare `str` or naive `datetime` —
   route-local validation helpers are the defect that rule replaces.
6. **Training objectives own evidence interpretation.** Capture Facts once,
   then let each exporter interpret only the evidence appropriate to its
   training objective. Every training row names one Evidence recipe and
   preserves the source of every label, eligibility decision, and Reward. Do
   not extend the legacy mixed DPO/SFT behavior; ADR 0011 defines its
   replacement.

## House Rules

- **Absent, never guessed at.** Anything unknowable is omitted visibly and
  logged — never fabricated (`verification_command`, manifest entries, diffs,
  judgments). In Derivations and projections, ineligible inputs are skipped
  **and counted** under the module's closed skip-reason vocabulary; capture
  and Derivation fail-soft paths additionally log-and-degrade rather than
  raise ([Fail-soft](CONTEXT.md#fail-soft)). Never silently dropped, never emitted
  half-formed.
- **Bump `policy_version`** when tuning any policy knob; the Provenance
  stamp is what keeps pre- and post-change datasets distinguishable.
  (`DPOPolicy`/`SFTPolicy`/`MirrorPolicy` carry no version field — a known
  gap; call such tunings out in the PR description.)
- **Every new Derivation ships determinism tests**: same Facts → identical
  output, and shuffled ingest order → identical output.
- **Climb before you build.** In order: does it need to exist at all, does
  this repo already have it, stdlib, a native platform or DB feature, an
  already-installed dependency, one line — then the minimum code that
  works. Stop at the first rung that holds. No interface with one
  implementation, no factory for one product, no config for a value that
  never changes, no new dependency for what a few lines cover. The ladder
  shortens the solution, never the reading: trace the real flow first, then
  climb.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sediment-ai/sediment](https://github.com/sediment-ai/sediment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
