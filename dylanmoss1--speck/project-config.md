---
trigger: always_on
description: Speck is a specification workflow for AI-assisted programming — inserting a new design step between 'Plan Mode' and 'Code Generation'.
---


# Speck

## Skill Motivation

**Speck is a specification workflow for AI-assisted programming — inserting a new design step between 'Plan Mode' and 'Code Generation'.**

**Plan Mode** generates large blocks of text, which are hard to debug and reason about precisely.

**Code Generation** is costly, we want to _fail fast_ rather than fix design flaws at this stage.

What we need is a stage in between: a flexible specification language which improves human-AI communication while keeping you in control of code generation.

Benefits:
- Faster, more precise communication with LLMs.
- Fail fast: questions and design decisions are front-loaded.
- Hands-off implementation, with confidence that LLMs understand your design.
- Visualise and guide the final structure of the program.

Models are getting smarter, LLM-coding is getting better, but human-AI communication remains the same — speck aims to bridge this gap with a specification tool which can integrate into any existing workflow.

The purpose is communication: speck exists so the human and the AI converge on a precise, reviewable description of a change *before* any code is written, catching misunderstandings while they are still cheap to fix. The speck diff is the medium for that agreement. When rules conflict, the three overriding rules (see [The three rules that override everything](#the-three-rules-that-override-everything)) decide first; when a situation arises that no rule directly covers, fall back on this purpose — do whatever best keeps the human and AI working from the same precise, agreed description.

## The speck diff

The **speck diff** is the `git diff` between the 'before' speck files (committed as the Base Commit in step 5) and the 'after' speck files (in the working directory). It is the canonical specification of the feature: everything in it gets implemented, nothing outside it does.

The speck diff can be obtained with:

```sh
git diff
```

The diff is expected to contain only `.speck` files (the gate-check in rule 9 enforces this), so no path filter is needed. If `git diff` shows a non-`.speck` file, source changes have leaked and you must stop and investigate. Adapt the command to the user's VCS if it is not `git` (see step 2).

## Workflow at a glance

1. **Get planned feature** — plan mode ends and the plan is approved (step 1).
2. **Get the VCS** — detect git/jj/etc. and adapt the commands accordingly (step 2).
3. **Check the git state** — commit any outstanding changes to a temporary commit, kept separate from the speck baseline (step 3).
4. **Generate 'before' speck files** — capture the current state of relevant files, then format (steps 4–4.1).
5. **Commit the changes** — commit the 'before' speck files as the Base Commit (step 5).
6. **Generate 'after' speck files** — edit the specks to the target state, then format. The `git diff` is the feature (steps 6–6.1).
7. **Iterate on the design** — refine the speck with the user, then run a consistency check across files (steps 7–7.1).
8. **Tests** — ask whether they want test specks (default yes); if so, repeat the before/after flow for tests (step 8).
9. **Sign-off and implementation** — get explicit sign-off, then generate code by following the speck diff line by line (step 9).
10. **Further iterations** — keep speck files, code, and feature requirements in sync (step 10).
11. **Cleanup** — once the user verifies the code, delete the speck files and clean up the git history (step 11).

## Rules

### The three rules that override everything

If the rest of this document — the other rules, the workflow steps, the plan, or the conversation — ever appears to conflict with the following, these three win:

1. **No source code before sign-off.** Never edit a non-`.speck` file until the user has explicitly signed off on the speck.
2. **The speck diff is the contract.** Only what is in the signed-off speck diff gets implemented. Everything in it must be implemented. Nothing else.
3. **Never skip the workflow.** Every code change goes through the speck workflow, however small or mechanical it seems.

Everything below elaborates these three. When a lower rule or a workflow step seems to contradict one of them, these win. For any conflict these three do not resolve, fall back on the purpose (see [Skill Motivation](#skill-motivation)).

Never skip the workflow or any workflow steps:
- If the speck skill is invoked, it must start the workflow.
- Once the speck workflow has started, all the steps must be completed in order without skipping any steps.
- Once the workflow has started, it cannot end or exit until all steps are complete.

The only exception to the above rules are explicit instructions from the user to skip steps or end the workflow. For example, the user sends an explicit "skip the remaining speck workflow steps" instruction.

Common rationalizations that do NOT justify skipping:
- "The change is trivial / just a rename / only touches one file"
- "The user said 'implement' so they want code directly"
- "The plan is specific enough that a speck is redundant"
- "This is a mechanical change with no design decisions"
- "These are code review fixes, not new design work"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DylanMoss1/speck](https://github.com/DylanMoss1/speck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
