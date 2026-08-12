---
trigger: always_on
description: Project instructions for AI coding agents that do not read
---

# AGENTS.md — Attune AI

Project instructions for AI coding agents that do not read
`.claude/` (Codex, etc.). Claude Code loads `.claude/CLAUDE.md`
instead. The shared, agent-agnostic core lives in
`content/collaboration/contract.md` and is projected into the
marked block below (and into `.claude/CLAUDE.md`) — edit the
master and re-run `scripts/project_collaboration_contract.py`,
never the block. Content outside the block is Codex-facing
orientation only.

## Overview

Attune AI — AI-powered developer workflows with cost optimization
and multi-agent orchestration. Python 3.10+, published on PyPI as
`attune-ai`. Stack: pydantic, anthropic SDK / claude-agent-sdk,
structlog, rich, typer.

```text
src/attune/
├── agents/            # Release agents, state persistence, recovery
├── workflows/         # AI-powered workflows (all SDK-native)
├── models/            # Auth strategy and LLM providers
├── meta_workflows/    # Intent detection, NL routing
├── orchestration/     # Dynamic teams, workflow composition
├── plugins/           # BasePlugin + register_mcp_tools() hook
├── telemetry/         # FeedbackLoop, UsageTracker
└── cli_router.py      # NL command routing
attune_redis/          # Redis plugin — bundled in the attune-ai wheel
```

<!-- attune:collaboration:start -->

<!-- generated from content/collaboration/contract.md - edit the master, then run scripts/project_collaboration_contract.py -->

## Cross-provider collaboration

### Principles

Every principle below names its enforcer — the ratchet, gate, hook,
or drift-guard test that makes it true without anyone remembering
it. A principle marked **aspirational** has no mechanical enforcer
yet: treat it as binding discipline, and treat adding its enforcer
as pickable work.

1. **The receipt beats the promise.** "Configured", "registered",
   and "exited 0" are claims; evidence of the user-visible behavior
   is the receipt. Delegated lanes declare their receipt type at
   launch and the lead re-runs receipts centrally.
   *Enforcer: **aspirational** (ruled discipline —
   `.claude/rules/attune/decision-routine.md` delegation receipts
   + this contract's Verification receipts section; no mechanical
   gate).*

2. **The code is the contract; spec text is a hypothesis.** Before
   executing any spec-named scope, grep the code for the property
   the phase targets and execute against THAT set.
   *Enforcer: **aspirational** (lessons-core rule; no gate can
   check intent — partially backstopped by drift guards below).*

3. **One source, projected — never hand-edited twins.** Skills,
   the collaboration contract, help pages, and docs feature pages
   are projections; edit the master and re-project.
   *Enforcers: `tests/unit/plugins/test_sync_agents_skills.py`
   (skills mirror), `tests/unit/scripts/
   test_project_collaboration_contract.py` (contract blocks),
   `tests/unit/lessons/test_core_mirror.py` (lessons core),
   `tests/unit/authoring/test_projection_drift.py` (authored
   projections) — all fail CI on drift.*

4. **Dangerous constructs are blocked, not discouraged.** No
   `eval`/`exec`, no unvalidated file paths, no bare `except`.
   *Enforcers: `src/attune/hooks/scripts/security_guard.py`
   (PreToolUse block on eval/exec), pre-commit detect-secrets,
   `tests/unit/gates/test_path_validation_gate.py` (AST scan —
   modules with write-capable file ops must reference a
   path-validation helper or hold an allowlist entry; seeded
   2026-07-29 with 35 vetted modules, ratchets shrink-only).*

5. **Coverage is a floor, not a goal.** Changed code carries
   ≥80% coverage; the local bar is 85%.
   *Enforcers: `codecov.yml` project+patch gates (80%),
   `tests/unit/ci/test_workflow_yaml.py::
   test_coverage_threshold_is_at_least_80` (the threshold itself
   is drift-guarded).*

6. **CI spends attention, never money.** Per-push/PR workflows run
   keyless (`ANTHROPIC_API_KEY: ""`); the real secret lives only in
   allowlisted, manually-dispatched or budget-capped jobs.
   *Enforcers: `tests/unit/ci/test_ci_spend_guard.py` (secret refs
   allowlisted, non-allowlisted assignments must be `""`),
   `tests/unit/ci/test_workflow_yaml.py`
   (timeouts/pinning/concurrency).*

7. **A failed gatekeeper fails the gate.** A security auditor that
   errors or goes missing fails the Security gate — absence is not
   a pass.
   *Enforcer: sentinel semantics pinned by
   `tests/unit/agents/test_release_prep_team_orchestration.py`
   (chair-ruled 2026-07-29).*

8. **Docs may not cite fiction.** A doc that names a symbol which
   no longer imports fails CI.
   *Enforcers: `doc-import-audit` CI job +
   `tests/unit/test_generated_doc_import_drift.py`; wiring claims
   checked by the `wiring-audit` job.*

9. **Identity and brand drift are ratcheted.** Legacy identifiers
   and retired framing cannot re-enter the tree.
   *Enforcers: G5 brand-drift pre-commit gate +
   `tests/unit/gates/test_brand_drift.py`,
   `tests/unit/gates/test_claim_drift.py`.*

10. **Context is budgeted.** Always-loaded rule bodies fit a
    byte budget; everything else is JIT-recalled via the index.
    *Enforcer: `tests/unit/rules/test_rules_residency_budget.py`.*

11. **Seats advise; the chair promotes; the lead integrates.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Smart-AI-Memory/attune-ai](https://github.com/Smart-AI-Memory/attune-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
