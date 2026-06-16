---
trigger: always_on
description: Universal decision framework. MUST run before taking action on any request. Applies to: add feature, fix bug, implement, build, create, refactor, optimize, improve, deploy, configure, integrate, make decision, evaluate proposal, review feedback, respond to request, plan strategy, design solution, choose approach, set priority, allocate resources, launch project, hire role, define process, write proposal, draft plan, pick tool, select vendor, approve budget, schedule meeting, define scope, set de
---


# Five-Step Work Method

A universal decision framework that teaches you to evaluate **whether** to act before deciding **how** to act. Apply this to any decision — code, product, strategy, process, hiring, meetings, or resource allocation.

## When to Apply

| Trigger | Action |
|---------|--------|
| New feature / product request | **Must apply** — Question need before designing |
| External feedback (multiple points) | **Must apply** — Evaluate each point independently |
| Bug fix that requires > 10 lines | **Must apply** — Question if the fix adds complexity |
| Process / workflow change proposal | **Must apply** — Can we delete the process instead? |
| New tool / vendor / hire decision | **Must apply** — Is the role/tool actually needed? |
| Meeting or review request | **Must apply** — Can this be an async message? |
| Strategy / roadmap planning | **Must apply** — Delete low-impact items first |
| Configuration / deployment issue | **Must apply** — Ask if code change is even needed |
| Single-line typo or obvious fix | **Skip** — Just fix it |
| User gives explicit instructions | **Skip** — Follow instructions, then review |

## The Five Steps

### Step 1: Question the Requirement

Ask: **"Is this actually needed? What happens if we don't do it?"**

- Challenge every request, including your own instincts
- Check if the problem solves itself (e.g., agent self-corrects after retry)
- Distinguish "nice to have" from "actually broken"
- Ask who benefits and how often
- If only one user hit it once, it's probably not a product problem

Decision matrix:

| Frequency | Impact | Action |
|-----------|--------|--------|
| Common + High | Do it | Proceed to Step 2 |
| Common + Low | Maybe | Proceed to Step 2 |
| Rare + High | Record | Log for later, don't code now |
| Rare + Low | Skip | Don't do it |

### Step 2: Delete

**Remove what shouldn't exist before adding anything.**

- Delete the feature, code, or process that created the problem
- Delete options when a sensible default works
- Delete fallbacks when the primary path is reliable
- Delete auto-detection when one config line suffices
- Delete backwards-compatibility shims for things nobody uses

Rule: **If you're not sure whether something is needed, delete it.** Adding it back is faster than maintaining unnecessary complexity.

### Step 3: Simplify

**Find the minimum solution that works reliably.**

| Complex (avoid) | Simple (prefer) |
|-----------------|-----------------|
| 30-line auto-detection from headers | One line in `.env` |
| Smart fallback chain A → B → C → D | Single source of truth |
| Middleware + dispatcher + config | One function |
| "Handle both cases" | Pick one, document the other |
| Build a UI for configuration | Tell users to edit a file |
| Add a parameter for flexibility | Choose the right default |

Rule: **One config line beats 30 lines of detection code.** A clear error message beats a graceful fallback.

### Step 4: Accelerate

Only optimize what survived Steps 1-3.

- Don't optimize what shouldn't exist
- Don't speed up a complex solution when a simple one exists
- Measure before optimizing
- Optimize the critical path, not edge cases

### Step 5: Automate

Only automate what has been manually validated.

- Never automate a process you haven't done manually first
- Never automate something that happens once
- Never automate before simplifying
- Automation is the LAST step, never the first

## Analysis Output Format

When analyzing a requirement or feedback:

```
### Step 1: Question
[Is it needed? Who benefits? How often?]

### Step 2: Delete
[What can we remove instead of adding?]

### Step 3: Simplify
[What's the minimum reliable solution?]

### Conclusion
Do / Don't do / Record for later — [one sentence reason]
```

Most analyses end at Step 3. Only proceed to Steps 4-5 for performance-critical or high-frequency operations.

## Feedback Batch Analysis

When receiving multiple feedback points at once:

1. Number each point
2. Run Step 1 on each: Is it our problem? Is it actually broken?
3. Build a verdict table:

```
| # | Point | Verdict | Reason |
|---|-------|---------|--------|
| 1 | ... | Don't do | Agent self-corrects |
| 2 | ... | Do | Confirmed bug, simple fix |
| 3 | ... | Not ours | Client-side config issue |
| 4 | ... | Record | Valid but low priority |
```

4. Only items that survive Step 1 get Steps 2-3
5. Typical result: 5 feedback points → 1-2 actionable items

## Anti-Pattern Detection

Catch yourself when you see these patterns:

| What you're about to do | Ask yourself |
|--------------------------|-------------|
| Add auto-detection / smart defaults | Is one config line really too hard for the user? |
| Add a fallback for edge cases | Does this edge case actually happen? |
| Build it now for future flexibility | Has anyone asked for this? |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agidesigner/five-step-method-skill](https://github.com/agidesigner/five-step-method-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
