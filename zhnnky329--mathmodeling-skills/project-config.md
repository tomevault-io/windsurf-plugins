---
trigger: always_on
description: - **The AI owns mechanical correctness; the human owns modeling judgment.** Assistant for the engineering discipline, not a solver or paper-writer. AI keeps numbers traceable, runs PoCs, audits, render-checks, enforces gates. The human chooses the method and says why, decides what a number means, judges results, frames assumptions, authorizes submission. A skill must never originate, on the human's behalf, a judgment a judge would grade.
---

# Core Philosophy

- **The AI owns mechanical correctness; the human owns modeling judgment.** Assistant for the engineering discipline, not a solver or paper-writer. AI keeps numbers traceable, runs PoCs, audits, render-checks, enforces gates. The human chooses the method and says why, decides what a number means, judges results, frames assumptions, authorizes submission. A skill must never originate, on the human's behalf, a judgment a judge would grade.
- Start from goals, objects, constraints, data, outputs, variables, relationships, and checkable conclusions.
- Do not start from model names or favorite techniques.
- Separate assumptions, observations, derivations, and validated conclusions.

# Workflow Discipline

- Parse the problem before classifying it.
- Classify the task before selecting methods.
- Generate a candidate method pool before committing to one method.
- Validate the method plan before writing code or paper text.
- Run multi-round experiments before locking in the final method.
- Keep every change minimal, traceable, and easy to review.
- **Change-propagation rule (P1)**: After modifying any file under `code/Qx/`, `methods/Qx/`, `results/Qx/reports/`, or `planning/`, grep the entire workspace for references to the changed artifact (symbol name, parameter name, file path, numerical value), list every file that now may be stale, and either update it in the same turn or flag it as STALE with recommended repair. Specifically:
  1. `grep -rn '<changed_identifier>' methods/ code/ results/ paper/ planning/` before claiming "done".
  2. If the changed file is under `code/Qx/` and `frozen_numbers.json` exists for that Qx, mark the freeze as stale in `results/Qx/reports/freeze_change_log.md`.
  3. After any multi-file change, run `consistency-auditor` for the affected Qx as an incremental check — do not defer to the final G6 run.

# Workflow Gates (G1 – G6)

Gates are not stages. A stage is "where I am"; a gate is "what I must satisfy to leave". The orchestrator evaluates each gate against an explicit `enter_condition / pass_criteria / fail_fallback` contract. See `.codex/skills/workflow-orchestrator/SKILL.md` for the full contract.

- **G1 PROBLEM_PARSED**: parse + classification exist.
- **G2 METHOD_VALIDATED** (load-bearing — method→code boundary): each candidate has a ≤30-line PoC + feasibility number. No PoC ⇒ not validated.
- **G2.5 METHOD_CHOSEN_BY_HUMAN** (human-decision gate, pilot): `methods/Qx/decisions/method-selector_modeler_decision.md` is `DECIDED` by the human with a non-empty, non-copied, evidence-citing rationale. `method-selector` only emits `[CANDIDATE — PoC PASS]` + an AI suggestion; the human commits the choice. `code_generation_allowed_Qx = G2 ∧ G2.5`. The orchestrator never fills the decision artifact or infers the choice from the AI suggestion.
- **G3 CODE_REVIEWED**: reviewer artifact exists at `code/Qx/reviews/qx_<lang>_review.md` with ≥ 5 explicit pass items.
- **G4.5 RESULTS_JUDGED_BY_HUMAN** (human-decision gate): before the freeze, the human renders the result verdict (`result-report-generator` → `qx_result_verdict`) and stability verdict (`robustness-checker` → `qx_stability_verdict`, citing a number). `[REJECTED]` archival fires only off a human tag. `freeze_allowed_Qx = G3 ∧ G4.5`. The why-this-method narrative is transcribed from the decision log, not re-authored.
- **G4 RESULTS_FROZEN** (load-bearing — results→paper boundary): requires G4.5 passed + package sign-off (`solution-package-builder` → `qx_package_signoff`). `frozen_numbers.json` exists and is newer than every source file; solution package sources all numbers from it; every "why we chose X" traces to a decision_id.
- **G5 PAPER_SECTION_READY**: section meets word-count floor; every numerical result has ≥ 3 discussion dimensions; every figure passes render-check.
- **G6 AUDIT_LAYER_PASSED**: all three independent audits PASSED (see "Independent Audit Layer" below).

Rules:
- Do not select methods before G1.
- Do not generate code before G2 (every candidate must have a PoC).
- Do not write numerical paper claims before G4 (numbers must be frozen).
- Do not hand a subquestion to the paper writer before the three critical rules are satisfied.
- Do not assemble the final paper before G6.
- A gate failure marks all downstream artifacts DIRTY — files at later stages are not automatically valid.

# Independent Audit Layer

`workflow-orchestrator` does NOT decide "completion". Three orthogonal auditors do — and any one of them failing blocks final assembly:

1. **consistency-auditor** → `paper/audits/cross_media_consistency_audit.md`
   Cross-media facts (numbers / file names / symbols / parameters) match across tex ↔ code ↔ frozen_numbers.json ↔ symbol_table.md.
2. **completeness-auditor** → `paper/audits/completeness_audit.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhnnky329/MathModeling-skills](https://github.com/zhnnky329/MathModeling-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
