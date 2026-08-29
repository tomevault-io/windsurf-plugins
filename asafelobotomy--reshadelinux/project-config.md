---
trigger: always_on
description: > **Template version**: 5.10.1 <!-- x-release-please-version --> | **Applied**: 2026-04-10
---

# Copilot Instructions — reshadelinux

> **Template version**: 5.10.1 <!-- x-release-please-version --> | **Applied**: 2026-04-10
> Living document — self-edit rules in §8.
>
> **Models**: each `.github/agents/*.agent.md` pins its model. Codex models are headless-only (no interactive prompts). See [model comparison](https://docs.github.com/en/copilot/reference/ai-models/model-comparison).
>
> **⚡ Critical Reminders** — every session, every task:
>
> 1. **Test** — use deterministic targeted suites during intermediate phases when available; run `bash tests/run_simple_tests.sh` before marking a full task done end-to-end (§2).
> 2. **PDCA** — Plan→Do→Check→Act for every non-trivial change (§3).
> 3. **Read first** — never claim or modify a file not opened this session (§4).
> 4. **Additive** — never delete existing rules without explicit user instruction (§8).

## §1 — Lean Principles

| # | Principle | This project |
|---|-----------|-------------|
| 1 | Eliminate waste (Muda) | Every line of code has a cost; every unused feature is waste |
| 2 | Map the value stream | Shell scripts run by user → Wine/Proton game prefixes updated with ReShade DLLs and shaders → enhanced visuals at game runtime |
| 3 | Create flow | User runs script → Steam library scanned → game selected → ReShade downloaded and installed into Wine prefix → shaders configured |
| 4 | Establish pull | Build only what is needed, when it is needed |
| 5 | Seek perfection | Small, continuous improvements (Kaizen) over big rewrites |

**Waste taxonomy** (§6):

- Overproduction · Waiting · Transport · Over-processing · Inventory · Motion · Defects · Unused talent

---

## §2 — Standardised Work Baselines

| Baseline | Value | Action if exceeded |
|----------|-------|--------------------|
| File LOC (warn) | 250 lines | Flag, suggest decomposition |
| File LOC (hard) | 400 lines | Refuse to extend; decompose first |
| Dependency budget | 6 runtime deps | Propose removal before adding |
| Dependency budget (warn) | 5 runtime deps | Flag for review |
| Test command | `bash tests/run_simple_tests.sh` | Must pass before the full task is done |
| Type check | `echo "no type check configured"` | Must pass before task is done |
| Three-check ritual | `bash tests/run_simple_tests.sh && echo "type-check: N/A"` | Run before marking complete |
| Integration test gate | INTEGRATION_TESTS=1 <!-- TODO: fill once known --> | Set to run integration tests |
| Max subagent depth | 3 | Stop and report to user |

---

## §3 — PDCA Cycle

Apply to every non-trivial change.

**Plan**: State the goal. List the files that will change. Estimate LOC delta. For non-trivial tasks that span multiple files or introduce new behaviour, write a brief requirements summary before coding. Realign before proceeding if that summary changes the plan.
**Do**: Implement. Write tests alongside code, not after.
**Check**: During intermediate phases or multi-part tasks, run the narrowest deterministic targeted suites for the touched paths when available. If the blast radius includes shared helpers, broad contract surfaces, or no reliable mapping exists, broaden aggressively. Run `bash tests/run_simple_tests.sh` only before marking the full user task complete end-to-end. Review output. Fix before proceeding.
**Act**: If baseline exceeded, address it now. Summarise what changed.

### Test Scope Policy

| Tier | Meaning | Use when |
|------|---------|----------|
| `PathTargeted` | Narrow deterministic checks mapped to touched paths | Default during intermediate work |
| `AffectedSuite` | Broader checks for shared helpers or broad contract surfaces | Path-targeted coverage is too narrow |
| `FullSuite` | Entire local test suite | Before marking the full task complete |
| `MergeGate` | Verified state required before merge, release, or final handoff | The change is ready to leave the working session |

- **Task complete** means the full user-visible task is finished end-to-end and the required verification has passed, not that one phase of a larger plan is done and not that one item in a multi-part TODO list is done.
- During intermediate phases, prefer deterministic path-based targeted suites tied to the files or directories actually touched.
- If the repo documents a targeted-test selector or phase-test command, use it to choose deterministic phase checks from changed paths instead of guessing the phase-time suite set manually.
- If multiple sub-parts are still in progress, do not treat a passing targeted subset as permission to declare the whole task complete.
- Broaden early when changes touch shared helpers, broad policy surfaces, parity mirrors, or any area without a reliable targeted test mapping.
- Final gate: before marking the full task complete, run the full suite with `bash tests/run_simple_tests.sh`.

### Structured Thinking Discipline

Before acting on any medium-to-complex task, apply this decision sequence to avoid
loop traps and wasted tokens:

1. **Frame** — state the problem in one sentence. If you cannot, the task needs
   decomposition before proceeding.
2. **Intent-Gate** — if the prompt is ambiguous, compound, or lacks scope, ask
  one clarifying question before acting. Never start execution on a prompt

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [asafelobotomy/reshadelinux](https://github.com/asafelobotomy/reshadelinux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
