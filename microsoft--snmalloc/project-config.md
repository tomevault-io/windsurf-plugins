---
trigger: always_on
description: **Complete the plan, then check in**: When a plan is approved, execute all
---

# Claude AI Guidelines for snmalloc

## Working Style

**Complete the plan, then check in**: When a plan is approved, execute all
steps to completion. Don't stop after each step for review. When you think
you're done, recursively apply all relevant principles from this file — check
each one, act on any that apply, then check again until no more principles
are relevant. Only then report completion and wait for feedback.

**Plans require discussion before implementation**: After devising a plan
(whether in plan mode or not), run the review loop (see "Mandatory review
checkpoints") before presenting it. Do NOT proceed to implementation until
the plan has been seen and explicitly approved.

**Store plans in PLAN.md**: Always write plans to `PLAN.md` in the repository
root so that context survives session boundaries. Update (not append to) the
file when the plan evolves. This is the single source of truth for what is
planned and what has been completed.

**Baseline the checkout before starting work**: Before beginning implementation
of any plan, verify that the current checkout builds and passes tests. Run the
build and test suite (per `.github/skills/building_and_testing.md`) and record the
results. If the baseline is broken, report the failures and stop — do not start
implementation on a broken base. Pre-existing failures that are not caused by
your changes must be acknowledged upfront so they are not confused with
regressions introduced by the plan. This establishes the ground truth against
which your changes will be measured.

**Every plan step must have a test gate**: Each step in a plan must produce
a testable result — a test, a build check, or a verifiable property — that
acts as the gate to the next step. Do not move to step N+1 until step N's
gate passes. This catches integration issues incrementally rather than
deferring all testing to the end. When writing a plan, structure it so that
independently testable components are implemented and verified first, and
later steps build on proven foundations.

**Mandatory review checkpoints**: At each of these points, run the full
review loop — spawn a fresh-context reviewer subagent, address findings,
spawn another fresh reviewer, repeat until a reviewer finds no issues. When
you disagree with a reviewer's finding, escalate — do not resolve disputes
unilaterally. Do not proceed past a checkpoint without a clean review.
1. **After devising a plan**, before presenting it for discussion. For plan
   reviews, adapt the reviewer prompt: instead of reading changed files and
   running tests, the reviewer should read the plan document, read existing
   code the plan references, verify assumptions about the codebase, and check
   for structural gaps (missing steps, naming conflicts, incorrect
   dependencies).
2. **After completing implementation and self-review**, before opening a PR.

The only exception: if you believe a change is truly trivial (a typo fix, a
one-line config change), ask for permission to skip the review. Do not decide
on your own that something is trivial enough to skip. When in doubt, run the
review.

**Go slow to go fast**: Before starting implementation work, identify and state
which principles from these instructions are most relevant to the current task.
This surfaces the right guidelines before they're needed rather than
rediscovering them after a mistake.

**Challenge me when the evidence says I'm wrong**: If a reviewer flags something
that contradicts what I said, or if you have concrete evidence that an
instruction is incorrect, raise it — don't silently comply. Present the evidence
and discuss it.

**Research findings belong in the plan**: If research or exploration surfaces
issues beyond the original task (inaccurate comments, dead code, related bugs),
include them as explicit plan steps — don't just mention them in the analysis
and move on. Anything worth noting is worth acting on or explicitly deferring.

**Self-review is part of done**: The recursive principle check described in
"Complete the plan, then check in" IS the self-review. It's not a separate
step — it's what "done" means. Never report completion without having done it.

**During reviewer loops**: At any point during the review loop — when fixing
findings, when unsure about a reviewer's suggestion, when making tradeoff
decisions — stop and ask. The automated review removes me as a gatekeeper, not
as a collaborator.

## Debugging Principles

1. **Logging is essential** - When debugging issues in allocator code, add tracing to identify the exact point of failure. Use `write()` directly to stderr/file rather than `printf`/`message` to avoid recursion through the allocator.

2. **New code is most likely at fault** - When tests fail after changes, assume the new code introduced the bug. Don't blame existing infrastructure that was working before.

3. **Baseline against origin/main** - Before assuming a system-wide issue, verify the test passes on `origin/main`. This confirms whether the issue is a regression introduced by your changes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/snmalloc](https://github.com/microsoft/snmalloc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
