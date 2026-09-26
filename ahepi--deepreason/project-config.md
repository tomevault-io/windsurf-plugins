---
trigger: always_on
description: DeepReason is a Popperian reasoning harness: it drives a provider model
---

# CLAUDE.md — operating DeepReason

DeepReason is a Popperian reasoning harness: it drives a provider model
(qwen3.5:397b on Ollama Cloud for every committed launch since
2026-08-25; glm-5.2 wrote most of the earlier corpus and remains a
registered choice) through conjecture–criticism cycles
over an append-only, replay-verifiable record. Everything meaningful is
TYPED — stops, denials, refusals, capability lifecycles — and the record
is the only admissible evidence about what a run did. Model prose is
never evidence; `log.jsonl`, `objects/`, `progress.jsonl`,
`run-status.json`, `REPLAY_VALIDATION.json`, and `verify_root` are.

## MANDATORY for every model working in Claude Code on this repository

**Never verify a review without the operator's explicit permission.**
Operator's words, verbatim (2026-09-05): "Don't not ever verify a review
without my explicit permission. Make it for every model working in Claude
Code and make it mandatory." Operational reading, binding on the monitor,
every executor window, every subagent and every audit or review window:
a review-kind task (an audit, a code review, a verdict on a delivered
branch, a monitor merge review) READS and REPORTS; it does not run the
full gate, `docs_verify`, the wheel smokes, a soak, a live call, or any
other verification instrument to "confirm" what it is reviewing, unless
the operator has said so for that task in so many words. Reproducing a
single cited check by its own command, where the review brief names it, is
reading; anything wider is verification and needs permission. This applies
to the monitor's post-merge test rings too: none without permission. The
incident: the 2026-09-05 spec-drift audit ran the whole 5,167-test gate for
69 minutes by a mistyped ring command in a read-only window. "I did not
mean to" is not permission. Recorded in the operator design laws below as
well; this block is here so it is the first rule read.

## Which workflow to use

Both families now begin with a MAP PREFLIGHT: resolve the work to
`DR-SUB-`/`DR-CON-`/`DR-SEAM-` ids from `docs/map/INDEX.md`, read the seam
before the subsystems, and read `INV-frozen-surfaces.md` before designing.
Record the ids in the tranche's first artifact so every later phase starts
from the same map.

Two skill families live in `.claude/skills/`. Route ALL substantive work
through one of them — they exist to prevent scope creep, missed steps,
and forgotten inputs:

- **Something is broken / suspicious** → `deepreason-orchestrator`
  (phases: dr-set-goal → dr-diagnose → dr-reproduce → dr-propose-fix →
  dr-implement-fix → dr-verify-outcome). Diagnosis comes from the typed
  record BEFORE code reading.
- **The operator suggests a change** → `dr-change-orchestrator`
  (phases: dr-capture-request → dr-spec-change → dr-plan-steps →
  dr-execute-step (one step per invocation) → dr-validate-change →
  dr-deliver-change). Authority is the operator's verbatim words,
  ledgered in REQUEST.md; every artifact traces to requirement numbers.

- **The operator asks what is broken / unused / out of date** →
  `dr-audit-orchestrator` (dimensions: broken, dead, docs-drift,
  spec-drift, goal-trace). Read-only: produces AUDIT_REPORT.md plus a
  ready-to-send fix prompt per finding; every verdict compares against
  `docs/AUDIT_BASELINES.md`. Rated for inexpensive models — every step
  is a command, a paste, or a baseline comparison.

Cross-routing: a defect found mid-change is PARKED, not fixed; a change
wished for mid-defect is PARKED, not implemented. One tranche, one goal.
The audit family never fixes anything anywhere — findings become parked
prompts for the other two families.

Cutting across both families, three skills:

- `dr-drive-harness` — the driving manual. Load it at the start of any
  session that runs, modifies, or diagnoses the harness: session
  preflight, the public CLI lifecycle, live-run ladder rules, and where
  to look before modifying (map order, frozen surfaces) or when
  diagnosing (record first). Also the routing index for both families,
  phase by phase, with the artifact each phase owns (§6).
- `dr-ask-the-right-question` — question discipline. Load it before
  acting on any ambiguous or terse operator message, whenever a phase
  says "stop and ask", and whenever evidence contradicts your
  expectation: it routes each question to the cheapest authority
  (record → framework → operator) and kills false forks before they
  spend operator attention.
- `pinker-write-for-readers` — communication discipline. Load it once
  per session, before the first message the operator will see. It
  REPLACED `dr-explain-to-operator` on 2026-09-03 at the operator's
  instruction (installed from their own upload; see Conventions below
  for what survives of the earlier rules). Its two companions,
  `pinker-clarity-workflow` (the router) and
  `pinker-teach-for-understanding` (for explaining a mechanism so the
  operator can use it, not just hear it), were installed with it.

## Third lane: treadle

Beside the two agent families above sits a third lane that is not an
agent workflow at all. `treadle` (vendored at `tools/treadle/`, config
at `/treadle.toml` and `/skills/`, board at `.swarm/` through
`scripts/swarm_gate.py`) is a DETERMINISTIC DRIVER: it walks READY

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AHepi/DeepReason](https://github.com/AHepi/DeepReason) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
