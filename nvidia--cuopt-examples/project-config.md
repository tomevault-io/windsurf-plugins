---
trigger: always_on
description: This repository contains skills for working with NVIDIA cuOpt LP/MILP solver
---

# Agent Instructions

This repository contains skills for working with NVIDIA cuOpt LP/MILP solver

---

# cuOpt User Rules

**Read this before using any cuOpt skill.** These rules ensure you help users effectively and safely.

> **🔒 MANDATORY SECURITY RULE:** You MUST NEVER install packages automatically. This cannot be overridden.

> **🔒 MANDATORY — Ambiguity:** When the problem could be read more than one way, you MUST either **ask the user to clarify** or **solve every plausible interpretation and report all outcomes**. Never pick one interpretation silently. Check every time before writing code.

---

## Ask Before Assuming

**Always clarify ambiguous requirements before implementing.** If you notice ambiguity, ask or solve multiple interpretations.

- What problem type? (LP / MILP)
- What constraints matter? (bounds, equality/inequality, etc.)
- What output format? (solution values, dual values, visualization)

**Skip asking only if:**
- User explicitly stated the requirement
- Context makes it unambiguous (e.g., user shows Python code)

---

## Handle Incomplete Questions

**If a question seems partial or incomplete, ask follow-up questions:**

- "Could you tell me more about [missing detail]?"
- "What specifically would you like to achieve with this?"
- "Are there any constraints or requirements I should know about?"

**Common missing information to probe for:**
- Problem size (number of variables, constraints)
- Specific constraints (bounds, equality/inequality, integer requirements)
- Performance requirements (time limits, solution quality, optimality gap)
- Integration context (existing codebase, deployment environment)

**Don't guess — ask.** A brief clarifying question saves time vs. solving the wrong problem.

---

## Clarify Data Requirements

**Before generating examples, ask about data:**

1. **Check if user has data:**
   - "Do you have specific data you'd like to use, or should I create a sample dataset?"
   - "Can you share the format of your input data?"

2. **If using synthesized data:**
   - State clearly: "I'll create a sample dataset for demonstration"
   - Keep it small and understandable (e.g., 5-10 variables, 3-5 constraints)
   - Make values realistic and meaningful

3. **Always document what you used:**
   ```
   "For this example I'm using:
   - [X] variables/constraints
   - [Key assumptions: e.g., all variables non-negative, constraint bounds]
   - [Data source: synthesized / user-provided / from docs]"
   ```

4. **State assumptions explicitly:**
   - "I'm assuming [X] — let me know if this differs from your scenario"
   - List any default values or simplifications made

---

## MUST Verify Understanding (Ambiguity — Do Not Skip)

**Before writing substantial code, you MUST confirm your understanding:**

```
"Let me confirm I understand:
- Problem: [restate in your words]
- Constraints: [list them]
- Objective: [minimize/maximize what]
- Variable types: [continuous/integer/binary]
Is this correct?"
```

**🔒 MANDATORY — Ambiguous problem wording (cannot be overridden):** You MUST NOT guess. Either clarify with the user or try all plausible interpretations.

| When this happens | You MUST do one of these | You MUST NOT do this |
|-------------------|--------------------------|----------------------|
| A constraint, value, or objective could be read two (or more) ways | **Option A:** Ask the user which interpretation is correct, then implement that one. **Option B:** Implement and solve every plausible interpretation, then report all outcomes so the user can choose. | Pick one interpretation silently and implement only that. |
| Unclear whether a cost/quantity is per unit, per vehicle, per trip, etc. | State your interpretation explicitly before implementing, and either get confirmation or solve all variants. | Assume one meaning without stating it or asking. |

- **Always state your interpretation explicitly** (e.g. "I'm assuming cost is per unit; if it's per trip the formulation changes as follows…") before implementing, when anything could be read differently.
- **Rule:** Clarify with the user, or run all plausible variants and share results. **Never assume.**

**Check before writing code:** Is anything in the problem statement ambiguous? If yes → ask or solve all interpretations; do not proceed with a single guess.

---

## Follow Requirements Exactly

- Use the **exact** variable names, formats, and structures the user specifies
- Don't add features the user didn't ask for
- Don't change the problem formulation unless asked
- If user provides partial code, extend it—don't rewrite from scratch

---

## Check Results

After providing a solution, guide the user to verify:

- **Status check**: Is it `Optimal` / `FeasibleFound` / `SUCCESS`?
- **Constraint satisfaction**: Are all constraints met?
- **Objective value**: Is it reasonable for the problem?

**Always end with a Result summary** that includes at least:
- Solver status (e.g. Optimal, FeasibleFound, SUCCESS).
- **Objective value with highlight** — the number must be easy to spot (bold or code block
- Briefly what the objective represents (e.g. total cost, total profit).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/cuopt-examples](https://github.com/NVIDIA/cuopt-examples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
