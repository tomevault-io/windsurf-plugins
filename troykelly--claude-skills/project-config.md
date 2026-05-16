---
trigger: always_on
description: This file governs how Claude instances work on this repository. This is a Claude Code plugin containing skills for issue-driven development.
---

# Claude Code Development Instructions

This file governs how Claude instances work on this repository. This is a Claude Code plugin containing skills for issue-driven development.

## Foundational Principle

**Your training data is over two years old. You do not know current APIs, patterns, or best practices without research.**

This is not a suggestion. This is reality. Act accordingly.

## The Contract

Working on this repository requires:

1. **Research before any action** - No modifications, no new code, no assumptions without current information
2. **Slow, methodical work** - Speed is not a value here. Thoroughness is.
3. **No corner-cutting** - Any shortcut is a failure. Stop and redo.
4. **Verify everything** - Claims of completion require proof

If you cannot commit to these principles, do not proceed.

---

## Research-First Methodology

### The Rule

**Research before ANY action.** Before writing or modifying any skill, you must research:

1. Current Claude Code documentation
2. Current plugin APIs and patterns
3. Current skill authoring conventions

No assumptions from training data. Your knowledge is stale.

### Research Source Priority

When researching, use this order:

| Priority | Source | Method |
|----------|--------|--------|
| 1 | Official Claude Code documentation | Use `claude-code-guide` agent or fetch official docs |
| 2 | This repository's existing patterns | Read existing skills, understand conventions |
| 3 | Web search for current information | Search for recent changes, API updates, best practices |

### Research Protocol

Before ANY modification or creation:

```
1. STOP - Do not write anything yet
2. RESEARCH - Gather current information from all three sources
3. DOCUMENT - Note what you learned and how it applies
4. VERIFY - Confirm your understanding is current and correct
5. THEN PROCEED - Only now may you begin work
```

### Announce Your Research

At the start of any task, announce:

> "I am researching current Claude Code documentation, existing patterns in this repository, and current best practices before proceeding."

This is not optional. This is the protocol.

---

## Slow and Methodical Work

### What This Means

- Read before writing
- Understand before modifying
- Verify before claiming completion
- One logical step at a time
- No batch changes without individual verification

### Prohibited Actions (Corner-Cutting)

The following are **failures** that require stopping and redoing:

| Prohibited Action | Why It's Prohibited |
|-------------------|---------------------|
| Writing code without reading existing patterns first | You don't know what patterns exist |
| Assuming API behavior from training data | Your knowledge is stale |
| Skipping tests or verification | Unverified work is incomplete work |
| Making multiple unrelated changes in one commit | Cannot verify or revert atomically |
| Claiming completion without explicit verification | Claims require proof |
| Proceeding when uncertain without researching first | Uncertainty means you lack information |
| Copying patterns without understanding them | Cargo-culting creates fragile code |
| Rushing to show progress | Speed is not valued here |
| Batching updates instead of continuous progress | Hides problems and delays feedback |
| Modifying a skill without reading it completely first | You cannot improve what you don't understand |

### The Spirit of the Rule

These prohibitions catch known failure modes. But they are not exhaustive.

The spirit is: **if an action prioritizes speed over thoroughness, it is wrong.**

If you find yourself thinking "I can skip this because it's not explicitly prohibited," you have already failed. Stop and reconsider.

---

## Understand-Before-Modify Rule

### For Existing Skills

Before modifying ANY existing skill:

1. **Read the entire skill** - Every line, not just the section you're changing
2. **Summarize its purpose** - In your own words, what does this skill do?
3. **Identify integration points** - What other skills call this? What does this call?
4. **Understand the patterns used** - What conventions does it follow?
5. **Only then modify** - Changes must preserve intent unless explicitly asked to change it

### For New Skills

Before creating ANY new skill:

1. **Research current skill authoring patterns** - How are skills structured now?
2. **Read 2-3 similar existing skills** - What patterns do they use?
3. **Identify where it fits** - What category? What skills will it integrate with?
4. **Draft the structure first** - Get the skeleton right before filling in details
5. **Verify against existing patterns** - Does it match conventions?

---

## Verification Requirements

### Before Claiming Completion

You must verify AND explicitly document:

| Verification | How to Verify |
|--------------|---------------|
| Skill loads correctly | Test that the skill can be invoked |
| Follows established patterns | Compare against existing skills in this repo |
| Documentation is complete | All sections present, no placeholders |
| No stale assumptions | Every claim backed by current research |
| Integration points work | Referenced skills exist and are compatible |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [troykelly/claude-skills](https://github.com/troykelly/claude-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
