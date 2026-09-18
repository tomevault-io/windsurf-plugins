---
trigger: always_on
description: Hard rules for anyone (human or agent) working in this repo. Read fully before writing code.
---

# Abide — Agent Instructions

Hard rules for anyone (human or agent) working in this repo. Read fully before writing code.

## Hard requirements

- **The hook must never break the agent.** Every hook path exits 0, bounds its own time, and writes nothing to stdout but the JSON the host expects. A hook that hangs, crashes or chatters takes the user's whole coding session with it. This outranks every other goal in this repo: a check that did not run is a missed violation, and a check that wedged the agent is a bug report.
- **No secrets in this repo, ever.** The API key is read from the environment, from a `.env` at the repo root, or from `~/.abide/.env`, which `abide login` writes with owner-only permissions and which is the only file abide may ever write a key to. Never accept it as a flag — a flag lands in shell history and in CI logs — and never log it.
- **Verify before claiming done**: tests green, `pnpm build` + typecheck clean, and the end-to-end path exercised against live Jev with latency measured. A rule checker that has never checked a real diff is not done.
- **No AI attribution in commits or pull requests.** No `Co-Authored-By` line for a model, no "Generated with" footer, no session link, no bot listed as an author. Whoever opened the pull request is the author of every commit in it. Coding agents add these by default: `.claude/settings.json` in this repo turns them off for Claude Code, and if you use another tool, find its switch before your first commit. A squash merge carries a co-author trailer over from the branch, so check the merge box too.
- **Pass user-facing words through the humanizer before shipping them.** Anything a person reads — CLI output, rule instructions, README prose, error messages, the compile skill — goes through the [humanizer skill](https://github.com/blader/humanizer/blob/main/SKILL.md) first. Copy that reads as machine-written spends trust the tool then has to earn back.

# General

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
- Preserve intentional blank lines between methods and logical blocks. Do not compact whitespace to reduce line count or satisfy a file-size target; restructure the code instead if length is a problem.
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

## 5. Comment Sparingly

**Default to no comment. Code says WHAT; comments say WHY — and only when the WHY isn't obvious.**

- Don't narrate what the code does — the reader sees the `map`, the `switch`, the names. Restating them is noise.
- Comment only the non-obvious: a reason, a gotcha, a workaround, a constraint, a "looks wrong but isn't."
- If code needs a comment to be understood, first make the code clearer (rename, extract). Comment only what better code can't express.
- Delete comments that restate the adjacent line. One earned comment beats ten obvious ones.
- Every comment must earn its place: if you can't name what it tells the reader that the code doesn't, drop it.

# TypeScript quality — exhaustive, deterministic types

Quality software is built on types the compiler can enforce. These are non-negotiable in this codebase.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coldteadotai/abide](https://github.com/coldteadotai/abide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
