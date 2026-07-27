---
trigger: always_on
description: >-
---


# Debugging Delegation Protocol

This protocol works alongside the debugging techniques rule. That
rule covers *how* to debug; this one covers *when and how to delegate*
debugging to a sub-agent, and what to do with the results.

This document is read by both the parent agent (the one spawning)
and the sub-agent (the one debugging). Each section is labeled with
its audience. If you're a sub-agent spawned to fix a bug, skip to
[If You Are the Sub-agent](#if-you-are-the-sub-agent).

## Why Delegate (parent)

Delegate debugging to a sub-agent rather than doing it inline for
two reasons:

1. **Context window hygiene.** Diagnostic cycles produce noise --
   test output, debug prints, intermediate hypotheses, failed
   attempts. That noise accumulates in the parent's context window
   and degrades its ability to reason about the original task. A
   sub-agent's context is disposable; the parent only sees the
   result.

2. **Preventing revert spirals.** Without delegation, an agent can
   end up four levels deep: fix A unmasks B, fix B unmasks C, and
   now the agent has a complex stack of changes it only half
   understands and starts desperately wanting to revert everything.
   With delegation, each fix is an isolated unit of work. If the
   sub-agent fixes A and reports that B was unmasked, the parent
   spawns a fresh sub-agent for B with a clean slate.

## When to Delegate (parent)

When a test fails, first try to trace the execution path mentally
from the test input to the point of failure. If this produces a
high-confidence diagnosis -- you can identify the specific line or
condition that's wrong and articulate *why* -- fix it directly.

If the mental trace does not yield a confident diagnosis, **delegate
to a sub-agent.** Do not start speculative investigation in the
parent context.

Debugging sub-agents should be launched in the foreground by default
(`run_in_background: false`). The goal is context isolation, not
asynchronous parallelism. Background debugging sub-agents are only
appropriate when the user explicitly asks for parallel/background work
or when the parent has a concrete plan to monitor and stop them.

## What to Provide (parent)

Spawn a `generalPurpose` sub-agent with three things:

1. **Context:** What the code is supposed to do. Relevant files,
   modules, and recent changes. Enough for the sub-agent to
   understand the area without exploring the whole codebase.

2. **Symptoms:** The exact error output -- not paraphrased. Which
   test failed, what the expected vs actual behavior was.

3. **Repro:** The specific command to reproduce the failure (e.g.
   `./ct.sh test_name`). The sub-agent must be able to run it
   immediately.

The sub-agent has the debugging techniques rule in its context
automatically (it's always-applied). You do not need to restate
the debugging methodology, but you should remind it to follow
structured diagnosis and not speculate.

---

## If You Are the Sub-agent

You were spawned to fix a specific bug. You have the debugging
techniques rule in your context -- follow it. Use structured
diagnosis, not speculation. The parent gave you context, symptoms,
and a repro command. Start by reproducing the failure.

Do not leave long-running processes behind silently. If you start a
watcher, dev server, background test command, or any other process that
may keep running after your investigation, stop it before reporting back
or explicitly report the command, purpose, and cleanup status.

After you have a fix, **run the full test suite** before reporting
back. You need to know whether your fix introduced regressions,
unmasked other bugs, or whether there were already other failures
you didn't cause. Check every failing test: determine whether it
could have been caused by your changes or whether it fails
independently of them (revert your changes temporarily if needed
to tell the difference).

Report back with one of the following outcomes.

### Outcome 1: Fixed the bug (common case)

You identified the root cause, fixed it, and the failing test now
passes. Before reporting back:

- Remove all diagnostic output (search for the `DBG_` prefix).
- Run the full test suite and report the results.
- Report the root cause and the fix.

Your diff should be minimal and surgical. Every changed line should
trace to the root cause.

### Outcome 2: Test needed modification

Rarely, the test itself encodes a requirement that no longer
applies. You may modify a test, but you must justify it by
articulating exactly which **requirement** changed and why the old
assertion is wrong. "The code does something different now" is not
a justification -- it's a restatement of the bug.

### Outcome 3: Extensive rewrite required

If the fix turns out to require non-trivial restructuring -- not a
surgical bug fix but a significant rewrite -- **revert all your
changes** and report back with:

- The diagnosed root cause.
- Why a surgical fix isn't sufficient.
- A proposed approach for the rewrite.

The parent or user decides whether to proceed. You are scoped to
bug fixes, not redesigns.

### Outcome 4: Other failing tests

After fixing your assigned bug, the full test suite may have other
failures. These may be:

- **Unmasked by your fix:** your fix is correct, but it exposed a
  bug that was previously hidden.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Chia-Network/chia-gaming](https://github.com/Chia-Network/chia-gaming) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
