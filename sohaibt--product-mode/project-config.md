---
trigger: always_on
description: > For mixed PM + engineering teams working with Claude Code.
---

# CLAUDE.md Product-Led Collaboration Guidelines

> For mixed PM + engineering teams working with Claude Code.
> Biases toward *shipping the right thing*, not just shipping fast.

Inspired by [Andrej Karpathy's observations](https://x.com/karpathy/status/2015883857489522876) on LLM coding pitfalls, rewritten to cover the failure modes that bite product teams before a single line of code is written: wrong problem, wrong scope, invisible tradeoffs, and unmeasured outcomes.

---

## Pre-Flight Checklist

Before writing or requesting any non-trivial code, every box below must have an answer in the thread. If a box is empty, either fill it - or explicitly flag that we're proceeding without it and why.

- [ ] **Problem** : Whose pain are we solving, in one sentence?
- [ ] **Why now** : What changed? (evidence, trigger, cost of waiting)
- [ ] **Scope** : Smallest change that tests the hypothesis
- [ ] **Success metric** : The one number we expect to move
- [ ] **Reversibility** : One-way or two-way door?

---

## The Seven Principles

### 1. Frame the Problem Before the Solution

**Code is the answer to a question. Get the question right first.**

Rules:
- State the user, the problem, and the job-to-be-done *before* proposing a solution.
- If multiple user segments are affected, name whose need takes priority and why.
- Check for prior art in the codebase before designing new.
- Distinguish *symptom* ("button is slow") from *problem* ("users abandon checkout").

**Test:** A non-author can restate who we're helping and what changes for them.

---

### 2. Make Assumptions & Unknowns Visible

**Don't pick silently. Don't hide confusion. Surface what you don't know.**

Rules:
- List assumptions explicitly before coding. Mark each: *validated / assumed / unknown*.
- If the request is ambiguous, present 2–3 interpretations with tradeoffs - don't guess.
- Name confidence on non-obvious choices: *high / medium / low*.
- Push back when a simpler or different approach would serve the goal better.
- Stop on confusion. Name what's unclear and ask.

**Test:** A reviewer can point to each assumption and confirm "yes, we agreed on that."

---

### 3. Ship the Minimum Viable Change

**Smallest change that delivers real signal. No speculative scope.**

Scope rules:
- Cut to the smallest version that tests the hypothesis - not the full vision.
- No features beyond what was asked. No "while we're here."
- No abstractions for single-use code.
- No configurability, flexibility, or error handling that wasn't explicitly required.

Edit rules:
- Touch only what the request requires.
- Match existing style; don't refactor what isn't broken.
- Remove orphans *your* change created; leave pre-existing dead code alone (mention it, don't delete).

**Test:** Every changed line and every added feature traces directly to the stated problem.

---

### 4. Name the Tradeoffs

**Every decision costs something. Make the cost visible.**

For non-trivial choices, state:

| Dimension | Question |
|---|---|
| **Value** | What outcome does this unlock? |
| **Cost** | Time, complexity, ongoing maintenance |
| **Risk** | What breaks if we're wrong? Who pays? |
| **Alternative** | What did we consider and reject, and why? |

Rules:
- Don't frame options as "right vs wrong." Frame as "optimize for X vs Y."
- Call out irreversible costs (data migrations, public APIs, UX patterns users learn).
- If there's no tradeoff, say so explicitly - that's a signal the choice is obvious or the thinking is shallow.

**Test:** A stakeholder who disagrees with the decision can still articulate *why* we chose it.

---

### 5. Define Done by Outcome, Not Output

**"Merged" is not "done." Done is "the user can do the thing, and it works."**

Acceptance criteria must include:
- **Functional** : Tests pass; edge cases handled.
- **User-facing** : A real user flow completes end-to-end.
- **Operational** : It's observable in production (logs, errors, analytics firing).

Transform weak tasks into verifiable goals:

| Weak | Strong |
|---|---|
| "Add validation" | "Invalid inputs are rejected with a clear message; tests cover each case" |
| "Fix the bug" | "Failing test reproduces it; test passes after fix; no regression elsewhere" |
| "Refactor X" | "Behavior identical before and after (tests green both sides)" |

Rules:
- For multi-step work, write the plan as numbered steps with a *verify* step after each.
- If acceptance criteria can't be written, the work isn't ready to start.

**Test:** Someone else can check whether the work is done without asking us.

---

### 6. Instrument Before You Ship

**If we can't measure it, we can't tell if it worked.**

For every user-facing or behavior-changing release, define *upfront*:

- [ ] **North-star metric** : The one number we expect to move
- [ ] **Baseline** : Current value (from data, not vibes)
- [ ] **Expected direction & size** : e.g. +5% conversion, –20% latency
- [ ] **Time horizon** : When we check (7 days? 30?)
- [ ] **Guardrail metrics** : What must *not* get worse (error rate, adjacent funnels, cost-to-serve)
- [ ] **How we'll read it** : A/B test, before/after, cohort, qualitative

Rules:
- Ship instrumentation *in the same change* as the feature. Never "we'll add analytics later."

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sohaibt/product-mode](https://github.com/sohaibt/product-mode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
