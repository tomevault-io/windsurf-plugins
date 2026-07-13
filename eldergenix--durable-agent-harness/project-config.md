---
trigger: always_on
description: You are working on **durable-agent-harness (`dah`)**: a research harness for long-horizon
---

# AGENTS.md — Operating manual for agents in this repo

You are working on **durable-agent-harness (`dah`)**: a research harness for long-horizon
agents (verification loops, checkpoint/rollback, budget-aware planning, failure recovery)
plus an **ablatable memory system** evaluated on multi-session tasks. The deliverable is
reproducible evidence, judged by agent engineers at labs. Honest negative results are wins.

## Read order (once per session)

1. `GOAL.md` — mission, pre-declared hypotheses H1–H5, frozen metric definitions, Definition of Done.
2. `RULES.md` — non-negotiables R1–R15 and which machine enforces each.
3. `FEATURES.md` — what exists vs. what's planned (includes tracked known gaps).
4. `TASKS.md` — the ordered plan; find the first unblocked `[ ]` task in the active milestone.
5. `docs/JOURNAL.md` — what the last session did and what it left for you.

## Ground truth commands

```bash
source .venv/bin/activate                     # Python 3.11 venv at repo root
python scripts/quality_gate.py --fast         # lint + compile + governance tests (~seconds)
python scripts/quality_gate.py --strict       # + full pytest, import checks, results integrity
python scripts/rules_lint.py                  # just the rule checks, verbose per-rule output
pytest -q                                     # full test suite
dah-baseline / dah-ablation / dah-analyze     # experiment CLIs (M3+; console scripts from pyproject)
```

The **fast gate green** is the precondition for saying "done" on anything (R15) — the
`stop` hook runs it and will bounce you back if it's red. The **strict gate** closes
milestones.

## Repo map

```
src/dah/
  types.py            # Fact/Turn/Task/ContextItem/AssembledContext — shared data model
  budget.py           # Budget: hard charge + soft pressure() in [0,1]; no refunds ever
  checkpoint.py       # Checkpointer over Snapshotable components; deep, total snapshots
  telemetry.py        # Trace.emit — the ONLY way library code reports what happened
  verifier.py         # oracle-free ConsistencyVerifier -> Verdict (empty-recall, low-relevance)
  recovery.py         # RecoveryPolicy: ingestion retries + rollback-before-retry
  planner.py          # StaticPlanner vs BudgetAwarePlanner (allocation, focused retries)
  faults.py           # seeded FaultInjector — own RNG stream, rate 0 == no injector
  agents.py           # one toggleable Agent loop; BaselineAgent vs HarnessAgent configs
  memory/
    stores.py         # WorkingMemory / Compactor / EpisodicStore / SemanticStore
    manager.py        # MemoryConfig + MemoryManager + 10 PRESETS — the ablation surface
  env/
    tasks.py          # synthetic multi-session task generator (owns ground truth)
    multisession.py   # MultiSessionEnv — sole grader; frozen metrics on EpisodeResult
  providers/
    base.py           # Provider protocol + ModelResponse
    simulated.py      # deterministic context-degradation model (length decay × middle loss)
    openai_compat.py  # optional live-LLM adapter; sole sanctioned network egress (R2)
  experiments/
    config.py         # single source of truth for every number in RESULTS.md
    runner.py         # config -> tidy DataFrame studies; paired bootstrap (R4)
    run_ablation.py   # dah-ablation: memory matrix + noise/fill sweeps
    run_baseline.py   # dah-baseline: harness ladder + budget/tau/recovery studies
    analyze.py        # dah-analyze: figures + hypotheses.csv verdicts + summary.md
    gallery.py        # results/traces.md — three annotated, seed-reproducible episodes
    live_check.py     # manual live-endpoint smoke test (needs OPENAI_API_KEY)
scripts/              # rules_lint.py, quality_gate.py (governance machinery)
tests/                # pytest; test_governance.py enforces R5/R6/R7/R1 + hook behavior
results/              # GENERATED ONLY — never hand-edit (R3); `make reproduce` rebuilds all
docs/                 # DESIGN.md (methodology), RESULTS.md (findings), JOURNAL.md (log)
.cursor/hooks*        # guardrails — do not weaken without user approval (R13)
```

## Architecture invariants (violating these invalidates the science)

1. **The env owns truth.** Only `env/` and `experiments/` may read `required_fact_ids` or
   grade. Memory/planner/verifier/recovery must never see the answer key (R5 — the linter
   greps for the tokens; don't alias them to sneak past it, the test suite also checks
   behaviorally).
2. **The provider is blind.** `SimulatedProvider` rolls recall for every fact in context;
   it cannot favor required facts. A fact absent from assembled context has recall
   probability 0 — that asymmetry is what makes memory load-bearing.
3. **Same degradation model for every arm.** Memory policies win by *placing the right
   facts at accessible positions under a token budget*, never by changing accessibility
   parameters. If an experiment varies `AccessibilityParams`, it's a sweep axis, applied
   identically to all arms.
4. **Determinism end to end (R1).** Seeds in, identical bytes out. Every RNG is an
   explicitly seeded `np.random.default_rng`; RNG state goes into snapshots (copy the
   `Compactor` pattern: `self.rng.bit_generator.state` in `snapshot()`/`restore()`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eldergenix/Durable-agent-harness](https://github.com/Eldergenix/Durable-agent-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
