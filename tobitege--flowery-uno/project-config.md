---
trigger: always_on
description: **THIS IS NON-NEGOTIABLE. VIOLATION OF THIS RULE IS UNACCEPTABLE.**
---

# Agent Rules

## Agent Behavior

### CRITICAL: THE 5-ITERATION RULE (Communication Before Scope Changes)

**THIS IS NON-NEGOTIABLE. VIOLATION OF THIS RULE IS UNACCEPTABLE.**

**The Problem This Solves**: The AI often starts a targeted fix, then mid-execution decides "this is too complex, let me simplify/rewrite this" — and proceeds to rewrite several hundred lines of code the user never asked to touch. This creates massive diffs, introduces regressions, and destroys context.

**The Rule**: During multi-tool inference (not chat with user), if an approach isn't working after **5 consecutive attempts**, the AI MUST:

1. **STOP making unsupervised changes**
2. **Report what was attempted** and why it's not working
3. **ASK the user** before trying a different approach or expanding scope
4. **NEVER silently change direction** - approach/scope changes require user awareness

**Clarification (Scope of This Rule)**: This rule is about **unsupervised approach changes during the AI's own multi-tool execution**. It does **NOT** limit interactive back-and-forth with the user. If the user is engaged and responding, keep working, but still **explicitly communicate** any approach/scope change before doing it.

**Specifically FORBIDDEN behaviors**:

- Silently deciding "this code is messy, let me refactor it while I'm here"
- Expanding from a 10-line fix to a 200-line rewrite **without informing the user first**
- Switching fundamental approaches (e.g., Grid → Canvas, inheritance → composition) mid-execution **without asking**
- "Simplifying" code by rewriting large sections the user didn't request

**What this rule does NOT prevent**:

- Continuing to help when the user is actively engaged in conversation
- Discussing options and alternatives with the user
- Making changes the user explicitly requests or approves
- Asking clarifying questions
- **Fixing root causes in other files** — but TELL the user first (e.g., "The bug in `DaisyButton` is caused by logic in `DaisyControlExtensions`. I'll need to fix it there. Proceed?")

**Reconciling "Fix Root Cause" with "Minimal Changes"**:

These are NOT contradictory. "Fix root cause" means don't apply band-aids. "Minimal changes" means don't refactor unrelated code.

- ✅ **DO**: Fix the root cause, even if it spans multiple files — just mention what you're doing (e.g., "Fixing this in `DaisyButton.cs` and `DaisyControlExtensions.cs`")
- ✅ **DO**: Make the minimal change needed in each file
- ❌ **DON'T**: Refactor unrelated code "while you're there"
- ❌ **DON'T**: Expand a bug fix into an architecture overhaul without discussion

**When to pause and ask** (not for every file, but for significant scope changes):

- The fix grew from ~10 lines to ~200+ lines
- You're about to rewrite a core abstraction or change a fundamental approach
- The "fix" would touch files completely unrelated to the reported issue

**Rationale**: The user asked for X. Deliver X. If X seems hard and Y seems easier, **ASK** before switching to Y.

**What counts as an "attempt"**:

- Each code modification targeting the same issue = 1 attempt
- User feedback/approval resets the counter (user is now involved in the decision)
- Scope expansion without **informing the user** = immediate violation (no 3-attempt grace period)
- If the user is actively engaged in conversation, continue normally; the "3 attempts" threshold only applies to **unsupervised** approach changes.

---

### Subagents

- ALWAYS wait for all subagents to complete before yielding.
- Spawn subagents automatically when:
- Parallelizable work (e.g., install + verify, npm test + typecheck, multiple tasks from plan)
- Long-running or blocking tasks where a worker can run independently.
- Isolation for risky changes or checks

---

### CRITICAL: NEVER REVERT OR SWITCH APPROACHES WITHOUT ASKING

**THIS IS NON-NEGOTIABLE. VIOLATION OF THIS RULE IS UNACCEPTABLE.**

When a solution is not working as expected:

1. **DO NOT revert code** without explicit user approval
2. **DO NOT switch to a different approach** without explicit user approval
3. **DO NOT "try another thing"** - ASK FIRST what the user wants to do
4. **STOP and present options** - Let the user decide the path forward

**Examples of FORBIDDEN behavior:**

- NO: "The PNGs aren't loading, let me switch back to SVGs" (without asking)
- NO: "This approach has issues, I'll try a different one" (without asking)
- NO: "Let me revert this change since it's not working" (without asking)

**Required behavior:**

- YES: "The PNGs aren't loading. Options: A) Switch to SVGs, B) Fix PNG deployment, C) Something else. What do you prefer?"
- YES: "This isn't working as expected. Should I revert or try to fix it?"
- YES: "I see Issue X. Before I change anything, what's your preference?"

**Rationale**: The user may have additional context, may prefer to debug the current approach, or may want to test on other platforms first. Unilateral reversions waste time and destroy progress.

---

### "No Change" Is a Valid Outcome

When the user reports an issue or asks for investigation **without specifying a required change**:

1. **Investigate first** - Understand the current behavior and why it exists
2. **Assess whether a change is needed** - Sometimes the behavior is correct/intentional

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobitege/Flowery.Uno](https://github.com/tobitege/Flowery.Uno) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
