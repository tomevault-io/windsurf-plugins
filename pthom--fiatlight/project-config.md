---
trigger: always_on
description: When helping users with coding tasks, please follow these guidelines to ensure high-quality, maintainable code.
---

# CLAUDE.md — fiatlight

# Guidelines for AI-assisted coding

When helping users with coding tasks, please follow these guidelines to ensure high-quality, maintainable code.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.
- If a solution becomes complex (multiple cascading changes, need for workarounds), STOP and explain the difficulty. Present options rather than plowing ahead.

**If you encounter an API in the codebase which is awkward to use**
- Do not circumvent it with a hack. Instead, surface the issue and ask for clarification or improvement.
- The same goes for code smells or patterns that seem out of place. Don't just "make it work" - stop implementing,
  then communicate the underlying problem so it can be addressed properly in collaboration with the user.
- This is **important**: we follow a trajectory where the codebase needs to become better and better.
  So, when we encounter something that is awkward (a code, an architecture smell) and which makes our job difficult, it is a good time to surface it.
  Then discuss with the user on how it could be improved to make the task at hand better, as well as all future developments a maintenance.
  The user might discuss with you on how to improve the architecture (do provide some well thought advices), or he might decide to say that a workaround
  is the way to follow. But this needs to be discussed.


**Before implementing a solution, wait for the user to finish evaluating alternatives.**

**No whack-a-mole loops.** When hitting a second unexpected failure in a row on a hard problem (especially cross-platform builds, CI, toolchain issues): STOP fixing. Present the full picture of what's going wrong and why, and ask to examine the difficulties together before writing more code. Investigation time up front saves much more than it costs. Similarly, before bumping a dependency version, check changelogs/release notes for new features that could interact with existing build flags.


## 1b. Interaction Style

**Wait before acting.** Do NOT start implementing or proposing solutions before the user has finished evaluating alternatives or describing the problem. Wait for explicit go-ahead. When asked to analyze or review, produce analysis ONLY — not code changes.


## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 3b. C++/Python Porting

When porting between C++ and Python: use raw strings for multiline content, use Python naming conventions (snake_case). Verify API names exist before using them (check the `.pyi` stubs).

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## 5. Debug from data, not from theory

**A single well-placed log is worth ten speculative fixes.**

- Before changing code to fix a bug, observe the actual values at the suspected fault site. Don't assume what a function returns — log it.
- If your fix doesn't work on the first try, stop coding and add observation. A second failed attempt without new data means you don't yet understand the bug.
- Choose log sites that will distinguish between the leading hypotheses. If you have two competing theories, log the values that would differ between them.

## 5a. Stop after two


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pthom/fiatlight](https://github.com/pthom/fiatlight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
