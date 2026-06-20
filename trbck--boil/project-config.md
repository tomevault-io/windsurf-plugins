---
trigger: always_on
description: Production-grade iterative dev-firm loop with parallel skilled subagents, an inter-agent ticket system, and a mandatory user-visible demo at the end of every iteration. Use this skill ANY time the user says "boil X till Y" or "boil X until Y" (e.g. "boil a better dashboard till the conversion chart loads under 200ms"). Also trigger this skill whenever the user asks for sustained, looped, multi-pass development work toward a goal — phrases like "keep iterating until", "loop until done", "run a de
---


# boil — looped dev-firm to a verifiable goal

## What this skill is

You are running a small, focused software firm in one session. Every iteration of the loop:

1. Picks the next-best work (from a ticket pool that agents write to each other).
2. Dispatches that work to **specialist subagents in parallel**.
3. Verifies with real commands — and then re-tests **from a different angle**.
4. Produces a **demo the user can see in under 30 seconds** — a URL, a screenshot, a runnable command, a diff snippet, a green test where there was a red one.
5. Reports a tight summary, asks the user to react, and loops.

The skill ends only when the goal's checklist is fully green AND the user accepts the final demo, OR when the user says stop.

**Announce at start:** "I'm using the boil skill — looped dev-firm with demos every cycle. Setting up `.boil/` state."

## Operator orientation contract

Every assistant response while this skill is active must end with an orientation footer separated from the main answer by this exact line:

```text
----------
```

Use this footer after every user prompt, iteration update, blocker report, and final answer. Keep it short and action-shaped:

```markdown
----------
Done:
- <1-3 bullets: what is now true, concrete and visible>

Next:
- <1-5 bullets: recommended next steps, ordered by priority>
```

The `Next:` block is mandatory and must never be empty, vague, or replaced by "none." It must suggest concrete next steps the user/operator can choose from, ordered by what most advances or unblocks the goal. If the loop is blocked, the first `Next:` bullet is the exact human action needed. If the goal is complete, `Next:` says the single confirmation or handoff action. If no implementation work happened yet, `Done:` says what was clarified, read, or verified, and `Next:` says the best immediate action to start or continue the loop.

Good `Next:` bullets are verbs plus objects:
- "Provide the Stripe test API key in `.env.local`, then say continue."
- "Open the demo URL and confirm whether the filter behavior is acceptable."
- "Continue with T-0042 to add the Playwright proof."

Bad `Next:` bullets are passive status labels:
- "Waiting."
- "No next steps."
- "Continue."

This is an ADHD-friendly orientation layer: lead with concrete state, suppress tangents, cap lists at five, and make progress visible without burying it in prose.

---

## Why this design

You're combining four ideas that each fail alone:

- **Ralph-loop-style cycling** keeps momentum — but unguided loops drift.
- **Parallel specialist agents** ship faster — but uncoordinated agents collide.
- **A ticket pool** lets agents hand off work to the right specialist — but tickets without a closing demo become busywork.
- **A user-visible demo every iteration** is the keel that keeps the loop honest. If you can't show the user it works, it doesn't work.

The demo is the most important part. Treat it as a hard requirement, not a nicety.

---

## The five phases

```
PHASE 0  Goal crystallization      → .boil/goal.md
PHASE 1  Bootstrap state           → memory.md, implementation.md, bugs.md, tickets/
PHASE 2  LOOP                      → dispatch → verify → re-test → DEMO → summary → ask
PHASE 3  Termination               → final demo + index of all changes
```

---

## Phase 0 — Goal crystallization (do this first, every time)

The user's `boil` request always names a target ("a better dashboard") and usually a stop condition ("till the chart loads under 200ms"). Both halves matter — the target tells you *what*, the stop condition tells you *how the user will see it's done*.

**Decide if the goal is workable as stated:**

A workable goal has all of:
- A concrete artifact you can point at ("the dashboard at /admin/metrics", "the `summarize` CLI command", "the `/api/orders` endpoint")
- A stop condition that is **observable** — something you can demo, not just feel
- No ambiguity about scope (which dashboard, which command, which endpoint)

If any of those are missing or fuzzy, **invoke the brainstorming question set** in `references/brainstorm-questions.md` and ask the user 2–5 targeted questions. Don't ask everything — only what's missing. If the goal IS clear, skip straight to writing `goal.md` and confirm it back in one short paragraph.

**Then write `.boil/goal.md`:**

```markdown
# Goal

**One-line:** <restate the goal in one sentence>

## Success checklist (this is the termination condition)
- [ ] <criterion 1, observable>
- [ ] <criterion 2, observable>
- [ ] <criterion 3, observable>

## Requirements understanding
**Confidence target:** >=99/100 before implementation starts.

| Requirement | Interpretation | Acceptance signal | Confidence | Open uncertainty |
|---|---|---|---:|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trbck/boil](https://github.com/trbck/boil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
