---
trigger: always_on
description: >
---


# Agentic Code Reasoning

## Purpose
Reason about code behavior using structured semi-formal analysis without executing repository code.

This skill enforces a certificate-based reasoning process: you must state premises, trace concrete code paths with file:line evidence, and derive formal conclusions. You cannot skip sections or make unsupported claims.

## Modes
- `compare` — determine if two changes produce the same behavior
- `diagnose` — find the root cause of a bug in a small number of files
- `explain` — answer a code question with verified evidence
- `audit-improve` — review code for security, API misuse, or maintainability

Choose a mode before exploring files. If unsure, prefer `explain`.

### Mode selection guide
| Trigger | Mode |
|---------|------|
| "Are these two patches/implementations equivalent?" | `compare` |
| "Where is the bug?" / failing test / single defect | `diagnose` |
| "What does this code do?" / "Why does X happen?" | `explain` |
| "Is this code secure?" / "Review for issues" | `audit-improve` |

### Activation gates

Before selecting a mode, check whether this skill is appropriate for the task. **Do not activate** this skill when:

- The task requires **broad file enumeration** (e.g., "list all files that need to change for this refactor"). This skill is designed for deep analysis of a small number of files, not for wide-coverage listing.
- The task is a **large-scale structural change** spanning many files (e.g., directory reorganization, rename propagation across a monorepo).
- The expected output is a **flat list of files** rather than a reasoned diagnosis with evidence.

For `diagnose` mode specifically:

| Condition | Use `diagnose` | Do NOT use `diagnose` |
|-----------|---------------|----------------------|
| Root cause scope | Likely 1–5 files | Many files (10+) across the codebase |
| Task nature | Single defect, specific test failure, error trace | Broad refactoring, feature addition, structural reorganization |
| Expected output | Ranked root cause with file:line evidence | Exhaustive list of files to modify |
| Evidence style | Deep code path tracing | Directory-level pattern matching |

If the task does not fit `diagnose`, consider using the agent without this skill — unrestricted exploration may produce better results for broad enumeration tasks.

---

## Core Method
Apply this process in every mode. **Complete each section in order. Do not write a later section before completing earlier ones.**

When a certificate template exists for your selected mode (see the mode sections below), **use that template as your primary guide** — it is the concrete implementation of Steps 1–6 for that mode.

### Step 1: Task and constraints
Write a short task statement and list constraints (e.g., no repository execution, static inspection only, file:line evidence required).

### Step 2: Numbered premises
Before concluding anything, write numbered premises grounded in known facts.

```
P1: [fact about the task, inputs, or expected behavior]
P2: [fact about relevant files, tests, or specifications]
P3: ...
```

Do not treat guesses as premises. Every later claim must reference a premise by number.

### Step 3: Hypothesis-driven exploration
Exploration priority is not a fixed reading order; choose the next action by discriminative power — what unresolved uncertainty it resolves.
Before opening any file, write:

```
HYPOTHESIS H[N]: [what you expect to find and why]
EVIDENCE: [what supports this hypothesis — cite premises or prior observations]
CONFIDENCE: high / medium / low
```

After reading, record:

```
OBSERVATIONS from [filename]:
  O[N]: [finding with file:line]
  O[N]: [another finding with file:line]

HYPOTHESIS UPDATE:
  H[M]: CONFIRMED / REFUTED / REFINED — [explanation]

UNRESOLVED:
  - [remaining questions]

NEXT ACTION RATIONALE: [why the next file or step is justified]
OPTIONAL — INFO GAIN: [what uncertainty this action resolves; which hypothesis/claim it would confirm vs refute]
```

Steps 3 and 4 work together: Step 3 is your real-time exploration journal. Step 4 is the accumulated function-behavior record you build *during* Step 3 — **add a row to Step 4 each time you read a function definition in Step 3.** Do not reconstruct the table from memory after the fact.

### Step 4: Interprocedural tracing
Update this table **in real time during Step 3** — add each row the moment you read a function definition. Do not write this table all at once from memory.

For every function or method encountered on a relevant code path, record:

| Function/Method | File:Line | Behavior (VERIFIED) | Relevance to test |
|-----------------|-----------|---------------------|-------------------|
| [name] | [file:N] | [actual behavior after reading the definition] | [which test(s) and why this function is on the relevant path] |

**Rules:**
- Read the actual definition. Do not infer behavior from the name.
- Mark the Behavior column VERIFIED only after reading the source.
- If source is unavailable (third-party library), mark UNVERIFIED and note the assumption. Search for type signatures, documentation, or test usage as secondary evidence. Optionally probe language behavior with an independent script.
- Trace through conditionals, mapping tables, and configuration — not just the happy path.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KunihiroS/agentic-code-reasoning-skills](https://github.com/KunihiroS/agentic-code-reasoning-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
