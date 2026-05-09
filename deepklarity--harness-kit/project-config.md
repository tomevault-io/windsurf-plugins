---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Memory Policy

**Never use MEMORY.md files.** Do not create, read, update, or reference any `MEMORY.md` files in this repository. All persistent project context belongs in `CLAUDE.md` files (root and per-project). If you need to store learnings, use `/workflows:compound` to write to `docs/solutions/` instead.

## Git Safety: No Stash

**Never run ANY `git stash` command** — not `git stash`, `git stash list`, `git stash pop`, `git stash show`, or any other stash subcommand. This is a hard ban on the entire `git stash` family, including read-only variants. Multiple agents may work in parallel on the same repo; stash operations create invisible shared state that causes silent data loss across concurrent sessions. Use branches, temporary commits, or disk-anchored files instead. If you need to check for uncommitted changes, use `git status` or `git diff`.

## Design Principles

**Default First**: Every feature should work out of the box with sensible defaults. Configuration, agent assignments, and settings should be *suggestive* — provide a good default that the user can override, rather than requiring upfront configuration. The system picks reasonable choices automatically; the user intervenes only when they want something different.

This applies everywhere:
- **Odin agent assignments**: Cheapest capable agent is suggested automatically; user can reassign before execution
- **Configuration**: Built-in defaults → project-local config → explicit flags (each layer overrides the previous)
- **Workflow**: `odin run` works end-to-end with defaults; staged commands (`plan`/`exec`/`assemble`) let users intervene when needed

## Philosophy Grounding

**Read `odin/docs/philosophy.md` at the start of every session.** It defines the 19 core tenets that govern how this repo is built. Don't just read it — internalize it. Every design decision, code review, and architectural choice should be traceable to one or more of those tenets.

### Applying philosophy in practice

The tenets are not aspirational — they're operational filters. Use them actively:

- **Before writing code**: Ask "which tenets does this serve?" If you can't name one, question whether the work is necessary.
- **During code review**: Check for tenet violations. Code that works but violates a tenet (e.g., hardcoded pipeline stages violating *First Principles*, or AI-smelling output violating *No Slop*) is not done.
- **When making trade-offs**: The tenets resolve ambiguity. "Should I add a config option or pick a default?" → *Default First* + *Good Enough* say pick the default. "Should I build a custom solution or compose existing primitives?" → *First Principles* + *Modular Composition* say compose.
- **When designing features**: Run the design through the relevant tenets as a checklist. A new execution feature should satisfy *Determinism*, *Proof of Work*, *Async Heavy*, and *Pareto Observability* at minimum.

### Key tenets to keep top of mind

These are the tenets most frequently relevant to day-to-day work:

| Tenet | One-liner | Watch for violations |
|---|---|---|
| **No Slop** (#12) | AI output meets the same bar as human code | Boilerplate comments, filler docstrings, "AI smell" |
| **Default First** (#8 Good Enough + design principle) | Work out of the box, override when needed | Requiring config before anything runs |
| **First Principles** (#11) | No hardcoded stages — everything is a task | Special-casing what should be composable |
| **Proof of Work** (#4) | Every task has evidence | Changes without verification, "it should work" |
| **Pareto Delegation** (#2) | Cheapest capable agent | Using opus for haiku-tier work |
| **Human-First Legibility** (#13) | Optimize for human scanning | Dense JSON in user-facing output |
| **No Slop** + **Taste** (#12, #17) | Volume is easy; judgment is the filter | Accepting first-draft AI output without curation |

### Philosophy violations as code smells

Treat tenet violations the same way you treat code smells — they signal something is wrong even if the code technically works. When you spot one, flag it explicitly: "This violates tenet X because Y" and propose the fix. Don't ship code that violates the philosophy just because tests pass.

## Methodical Problem-Solving

**Think step by step. Observe before acting. Change one thing at a time.**

### Breadcrumb-First Exploration

**Before exploring the codebase for how a flow works, check `docs/breadcrumb_analysis/_INDEX.md` first.** It contains end-to-end workflow traces with FLOW.md (high-level), DETAILS.md (file/function level), and DEBUG.md (logs, search patterns, commands). The index maps common symptoms directly to the right debug doc. If a breadcrumb covers your area, read it before spawning any exploration subagents — it's cheaper and more accurate than re-discovering the same information.

When something is broken, unexpected, or unclear — resist the urge to jump to a fix. Follow this discipline:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deepklarity/harness-kit](https://github.com/deepklarity/harness-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
