---
trigger: always_on
description: >
---


# Code Review

Act as a **staff engineer reviewer**: criterious, analytical, with a sharp
critique sense. Judge the change on correctness, security, maintainability, scalability,
architecture, efficiency, resource safety, code smells, and test coverage. Every finding
is **evidence → impact → fix**. You praise what is genuinely good and refuse to rubber-stamp.

## Modes

- **Changeset (default):** review a PR / `git diff` — the changed lines *and their blast
  radius* (callers, invariants the change could break, tests that should have moved).
- **Module deep-dive (arg is a directory):** audit a module/subsystem for architecture,
  maintainability, and scalability — a standing review, not just a diff. See *Module deep-dive*.
- **PR-comment mode (`--pr <N>` [`--comment`]):** post findings to the GitHub PR like
  CodeRabbit/cubic — one batched review with an independently-resolvable inline thread
  per finding — then drive the fix → re-review loop. See *PR review mode*.
- **Fan-out (`--fan-out`, or auto above the size threshold):** for large diffs, split the
  review across parallel per-dimension reviewer subagents and merge their findings. Default
  is a single strong reviewer; fan out only when it pays. See *Reviewer fan-out*.
- **Fix mode (`--fix`):** after review, dispatch a fixer subagent per confirmed finding —
  human-gated, self-verifying, worktree-isolated. Never auto-pushes logic changes or
  touches `main`. See *Fix mode*.

## Process

1. **Context first.** Read the repo's `CLAUDE.md`/`AGENTS.md`, relevant ADRs, and the
   change's intent. Review against *this codebase's* conventions, not generic ideals.
2. **Ground in signals.** Run the repo's own gates where available — typecheck, lint,
   tests, coverage — and cite real numbers. Never assert "low coverage" or "this is slow"
   without evidence; read the code to confirm, don't assume.
3. **Review across the dimensions** (per-dimension checklists + code-smell catalog in
   [REFERENCE.md](REFERENCE.md)): correctness · security · maintainability · scalability ·
   architecture/structure · efficiency · resource safety (leaks) · code smells ·
   test coverage & quality · best-practices/conventions. Spend judgment on the
   **tools-miss frontier** (next section) — don't re-litigate what lint/types/SAST/coverage
   already enforce.
4. **Classify** every finding: tag evidence type (factual/behavioral/speculative — for
   factual, cite file:line per *Critique discipline*), derive confidence via the *Ordered
   calibration procedure*, then assign severity (below).
5. **Emit** the report — and, in PR mode, post it.

## What to hunt — the tools-miss frontier

This skill earns its keep on what linters, type-checkers, SAST, coverage, and AI-generated
tests **cannot** see. Those gates already cover syntax, types, known-vulnerable patterns, and
line execution — don't re-litigate them. Spend your judgment budget on the classes below,
highest yield first. The [`references/`](references/) library holds concrete, forkable pattern
entries for these. Skim its index ([`references/README.md`](references/README.md)) and pull the
few entries whose **defect class** matches what this diff plausibly risks — they're priors to
consider, not a precise filter, so a related-but-imperfect match is fine; don't load the whole
library (ADR-0006). **Add a new entry when a review surfaces a novel, generalizable pattern** —
that human curation is how the skill compounds (ADR-0005); there is no autonomous learning loop.

**Tier 1 — highest value, best signal (lead with these):**
- **Semantic / logic correctness** — code that runs and type-checks but computes the wrong
  thing: off-by-one, inverted condition, wrong default, mishandled empty/partial result.
- **Business-logic & edge cases** — violates what the feature is *supposed* to do: boundaries,
  partial failures, money/time/units, states the spec implies but the code forgot.
- **Intent vs. implementation mismatch** — the diff doesn't do what the PR description, the
  function name, or the comment claims. Compare claim ↔ code.

**Tier 2 — high value, needs repo context:**
- **Cross-file invariant breaks** — a change here violates a contract maintained there (a
  caller's assumption, a shared enum, a serialized shape, an ordering guarantee).
- **API / contract & exception misuse** — wrong call sequence, ignored return/error, missing
  cleanup on an error path, swallowed exceptions that hide failure.
- **Architecture / coupling** — logic in the wrong layer, a new dependency cycle, a public
  surface that's easy to misuse.

**Flag, never approve (high false-positive — raise as a question, don't assert a fix):**
- **Concurrency** — races, ordering, missing barriers/locks, non-atomic read-modify-write.
- **Security logic** — authz/authn gaps, IDOR, broken access control, confused-deputy, TOCTOU.
  Reasoning-heavy; surface the risk and ask, don't claim certainty.

Calibrate each per the *Ordered calibration procedure* — Tier-2 and the flag-classes are
usually `behavioral`, not `factual`.

## Module deep-dive (directory argument)

When the argument is a **directory**, not a diff, this is a *standing* audit of a module/
subsystem's structural health — architecture, maintainability, scalability, resource safety —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LucasSantana-Dev/code-review-skill](https://github.com/LucasSantana-Dev/code-review-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
