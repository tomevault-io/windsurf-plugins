---
trigger: always_on
description: Temper reference: fix
---



# Fix: Root Cause Analysis + Structured Fix

**Goal:** Investigate root cause, write a regression test that proves the bug, implement the minimal fix, review the fix, validate. Never guess — investigate first.

## Active Skills

- **Context Engineering** — load hierarchical context at stage start (rules → arch → source → errors, under 2K lines/task)
- **Temper Core** — stack detection, pack resolution, quality gates
- **Source-Driven Development** — before writing framework-specific code: detect installed version → fetch current docs → cite sources → surface API conflicts

## Usage

```
/temper:fix "users get 500 error on checkout"
/temper:fix "JIRA-123"
/temper:fix "#456"
```

## Bug: $ARGUMENTS

## Execution

### Context Loading

This stage may run in two modes:
- **Standalone** (`/temper:fix`) — the command file (`.claude/commands/fix.md`) acts as the orchestrator, running this methodology across 4 Agent subprocess stages (RCA -> Fix -> Review -> Check)
- **Agent subprocess** (from `/temper`) — starts with CLEAN context, only loads what's listed below

**This reference file describes the methodology** for each stage. The **command file** handles the orchestrator routing, stage gates, and state management. When running as a subprocess, only the steps relevant to the current stage are executed.

**Subprocess mode override:** When running as an Agent subprocess, do NOT show AskUserQuestion gates or clear context. Return the summary to the orchestrator. The orchestrator handles all gate decisions and context transitions.

In both modes, the fix methodology is identical.

**Context loading strategy:** Apply the context-engineering skill for hierarchical loading (rules -> arch -> source -> errors, under 2K lines/task). The file list below specifies WHAT to load; the skill specifies HOW and WHEN.

### Step 1: Detect Input Type

Same as /temper:plan Phase 0 — detect Jira, GitHub, or direct description.

**Extract from ticket/description:**

- Symptom (error message, wrong behavior, crash)
- Trigger (which user action, endpoint, data)
- Reproducibility (always, intermittent, specific conditions)
- When it started (recent deploy, specific date, always existed)

### Step 1.5: Load Enabled Packs

Read `.claude/temper.config` to get the list of enabled packs. For each enabled pack, load `.claude/packs/{pack}/rules.md`.

If stack detected, also load `.claude/packs/stacks/{detected-stack}.md` for stack-specific patterns.

These rules are applied during:
- **RCA** — check if the bug violates any pack rules (e.g., security pack: was input validation skipped?)
- **Fix approach validation** — validate the proposed fix doesn't introduce new pack violations (Step 3.5)
- **Fix implementation** — ensure the fix doesn't introduce new pack violations
- **Validation** — `/temper:check` validates against all enabled pack rules

```
Loading enabled packs: quality, tdd, security, git
Loading stack-specific rules: {detected-stack}
  quality: {N} BLOCK, {N} WARN rules
  tdd: {N} BLOCK, {N} WARN rules
  security: {N} BLOCK, {N} WARN rules
  git: {N} WARN, {N} SUGGEST rules
  {stack}: {N} patterns to follow
```

### Step 2: Root Cause Analysis (via Explore subagent)

Launch an Explore subagent:

```
Investigate a bug and find the root cause. Understand WHY it happens, not just WHERE.

BUG DESCRIPTION:
{ticket content or user description}

MULTI-HYPOTHESIS INVESTIGATION:

1. LIST ALL PLAUSIBLE CAUSES (max 5):
   Based on symptom, generate hypotheses with confidence + evidence:

   | # | Hypothesis | Confidence | Evidence |
   |---|------------|------------|----------|
   | 1 | {cause}    | HIGH/MED/LOW | {why you think this} |
   | 2 | {cause}    | HIGH/MED/LOW | {why you think this} |
...

SKIP CONDITION: If only ONE plausible cause exists OR you have an exact stack trace pointing to a specific line, proceed directly to Step 2 investigation. Otherwise, continue with multi-hypothesis approach.

2. INVESTIGATE TOP HYPOTHESIS:
   - Start with highest confidence hypothesis
   - SEARCH for related code (error messages, stack traces, domain keywords)
   - TRACE the execution path (entry point → ... → failure point)
   - Write a quick regression test to CONFIRM/DENY the hypothesis

   MCP CALL CHAIN (during execution path tracing):
   If code-review-graph MCP server is available and tools.mode is not heuristic-only:
   1. Call query_graph_tool with the suspected function name:
      - Request both callers (who calls this function) and callees (what this function calls)
      - Returns full call chain: entry point → intermediates → failing function
   2. Call get_affected_flows_tool for user-facing flows through the suspected function:
      - Returns which API endpoints / user actions reach the failing code
      - Identifies blast radius in terms of user-facing behavior
   3. Evidence: [PROVEN] call chain (AST-level, mechanically verified)
   If MCP unavailable:
      Use grep-based call chain tracing → [HEURISTIC]

3. IF HYPOTHESIS DENIED:
   - Fall back to next highest confidence hypothesis
   - Repeat investigation
   - Max 3 hypothesis attempts before asking user for more context

4. CHECK common root causes:
   Off-by-one, null/undefined, wrong operator (= vs ==, && vs ||),
   race condition, type coercion, incorrect ordering, missing switch case,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [galando/temper](https://github.com/galando/temper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
