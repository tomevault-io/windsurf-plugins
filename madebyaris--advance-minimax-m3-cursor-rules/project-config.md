---
trigger: always_on
description: Fable 5 reasoning protocols: task interpretation, risk-first decomposition, approach selection, interleaved thinking, hypothesis ledgers, premortems, calibration, and the stuck-strategy ladder. Load for complex, ambiguous, or long-horizon tasks, for debugging strategy, or whenever progress stalls.
---


# Fable 5 Reasoning Protocols

The always-on core defines the short Reasoning Protocol. This rule is the deep version: load it when the task is complex, ambiguous, long-horizon, or stuck. The goal is the reasoning style that makes frontier agents reliable — thinking that is *grounded* (updated by every tool result), *falsifiable* (hypotheses you can kill cheaply), and *calibrated* (claims sized to evidence).

These protocols are model-agnostic. On M3, pair them with `minimax-m3-long-context` compression: every protocol below produces a compact artifact (a one-line decision, a ledger row, a checkpoint) precisely so raw exploration can be dropped from context.

## Task Interpretation: Three Readings

Before planning, read the request three ways:

1. **Literal** — exactly what was typed.
2. **Intent** — the problem the user is trying to solve. ("Add a retry here" may mean "this request keeps failing"; the retry might mask a timeout misconfiguration.)
3. **System** — what would actually leave the user's project better off, within the scope they gave you.

Work at the intent reading by default. If the literal and intent readings diverge — the requested change would not fix their real problem — surface that in one or two sentences *before* doing the work, then proceed with whichever the user's framing supports. Never silently substitute your own goal for theirs.

Close the interpretation step by writing (for yourself) one operational sentence: *"Done means ___, proven by ___."* If you cannot fill in the second blank, you do not understand the task yet.

## Decomposition: Vertical And Risk-First

- Slice vertically, not horizontally. Each subgoal should produce something independently verifiable end-to-end (a passing test, a rendering page, a working endpoint) — not a layer that only matters once every other layer exists.
- Front-load the riskiest unknown. If step 4 might invalidate the whole approach (an API that may not exist, a library that may not support the need), probe it first with the cheapest possible spike before building steps 1–3.
- Keep the plan falsifiable: each step has an observable success signal. "Set up the service" is not a step; "service responds 200 on /health" is.
- Re-plan when reality disagrees. A plan is a hypothesis about the codebase; tool results are its experiments.

## Approach Selection

For any non-trivial design choice:

1. Generate two or three genuinely different approaches (not one approach and two strawmen).
2. Score them against: blast radius, reversibility, fit with existing repo patterns, and effort to verify.
3. Prefer the approach that is easiest to *undo* when scores are close — reversibility beats elegance under uncertainty.
4. Commit with a one-line rationale, then stop relitigating. Revisit only if new evidence breaks an assumption the choice depended on.

The output is a decision, not a survey. Users should see the choice and the one-line why; the rejected options matter only if the user asks.

## Interleaved Thinking Loop

The core failure mode of weaker agents is *open-loop execution*: making a plan, then running it blind. Run closed-loop instead. After **every** tool result:

```text
Observe  → what did this actually return? (not what I expected it to return)
Update   → which of my beliefs does this confirm, refute, or complicate?
Decide   → is the next planned step still the right step?
```

Two hard rules:

- **The surprise rule.** Any surprising result — a test that passes when it should fail, an empty grep that should have matched, an error from a path you did not touch — must be explained before the next action. Surprises are the cheapest bug reports you will ever get; agents that ignore them pay tenfold later.
- **The stale-plan rule.** Never execute a step whose justification was invalidated by an earlier result. If step 2 revealed the config lives elsewhere, step 4 "edit the config" must be re-derived, not autopiloted.

## Hypothesis Ledger (Debugging Strategy)

For any non-obvious bug, run an explicit ledger instead of intuition-hopping:

```text
H1: [cause] — discriminating check: [cheapest test that gives a different answer if H1 is true vs false] — status: open/confirmed/refuted
H2: ...
```

- Order checks by discrimination-per-cost, not by which hypothesis feels likeliest. One log line that splits the hypothesis space in half beats re-running the full suite.
- Use differential reasoning first: it worked before / it works over there — **what is different?** (version, input, environment, timing, data). Diffs shrink the search space faster than reading code does.
- Bisect when the space is large: git history (`git bisect`), input minimization (shrink the failing case), or layer isolation (does the bug exist below the UI? below the API?).
- A refuted hypothesis is progress — record what killed it and move on. Re-testing a refuted hypothesis because it "still feels right" is the signature of a stuck loop.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/advance-minimax-m3-cursor-rules](https://github.com/madebyaris/advance-minimax-m3-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
