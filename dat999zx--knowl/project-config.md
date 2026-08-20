---
trigger: always_on
description: Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.
---

# AGENTS.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

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

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

<!-- KNOWL_PROJECT_MEMORY -->
## Knowl Project Memory

### Required workflow

1. For every project-specific request, call `knowl_query` before repository files or commands, using the words that name the subject: another on-subject term retrieves better, an off-subject one retrieves worse, so do not pad the query and do not trim a real term to shorten it.
2. Skip a new query only when directly relevant active lifecycle context, a same-request query, or manual `knowl_task_start` relevant memory already answers it.
3. Use a relevant active hit immediately. Inspect files only after a miss, conflict, stale/low-confidence memory, or explicit verification request.
4. Query again before switching to a distinct subtask or project area, and before choosing how to build something new — existing tooling and pipelines are project knowledge, and in a linked workspace they often live in a sibling repo, so leave method queries unscoped.
5. Store or update durable knowledge during work and before the final answer — verified findings, stated intent (goals, plans, direction the user voiced) stored as goals with user_stated provenance even while unsettled, and resolved diagnoses stored as skills when the cause will recur (an environment quirk, a config trap — not a typo). The test: could a fresh session recover this from memory alone? Never store raw transcripts, secrets, or transient debugging noise.
6. Listed but not callable is not unavailable. A host may namespace the tools and withhold their schemas until asked, so load the schema for the name your host lists and call it — where names are namespaced from the server key, `knowl_query` appears as `mcp__knowl__knowl_query`. Stop and tell the user only when the tools are genuinely absent, or when every call fails; never silently bypass Knowl.

### Lifecycle modes

- **Automatic host lifecycle:** verified hooks own bootstrap, capture, checkpoints, and finalization. Never call `knowl_task_start`, `knowl_task_checkpoint`, `knowl_task_finish`, or `knowl_session_finish` for that hook-owned session.
- **Manual work loop:** without verified hooks, use `knowl task run` for one bounded command. For resumable work, start once, checkpoint meaningful milestones/blockers with the returned task ID, and finish exactly once after verification. The start result satisfies the initial focused lookup.

Casual conversation, a single memory lookup, and trivial non-resumable work do not create a manual task loop.

### Complete MCP tool routing

| Group | Tools | Routing |
| --- | --- | --- |
| Focused retrieval | `knowl_query` | Default first call for a specific project request and again when switching areas. Use the words that name the subject, without padding, and omit category unless certain. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dat999zx/knowl](https://github.com/dat999zx/knowl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
