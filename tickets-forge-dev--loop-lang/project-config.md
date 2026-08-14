---
trigger: always_on
description: This file teaches an AI assistant (Claude Code, Copilot, Cursor, etc.) how to write
---

# AGENTS.md — authoring LoopFlow (`.loop`) flows

This file teaches an AI assistant (Claude Code, Copilot, Cursor, etc.) how to write
**LoopFlow** flows. When a user asks you to design a staged, self-correcting, or human-gated
coding workflow — "set up a loop to fix X", "turn this epic into a pipeline", "automate
this multi-step task" — author a `.loop` file using the grammar below, then let the user
run it with `loop-run run file.loop`, or in Claude Code via `/loopflow run file.loop`.

A `.loop` declares a goal, the context to read, the allowed actions, the verification
(`done when`), and the stopping rules. This file is the complete grammar plus the
authoring procedure.

## When to write a `.loop`

Write one when the work is a *repeatable, verifiable* loop or a sequence of them:
bug fixes with a test, refactors gated by a check, an epic broken into stories, a
migration with a verification step. Don't write one for a one-off question or a trivial
edit — just do those directly.

Before building a loop, run the four-condition test — build one only when all four hold:

1. **Does the task repeat?** A one-time task is just a normal prompt.
2. **Is there a clear definition of "done"?** You must be able to verify completion — a
   `done when` predicate (a test, a command, or a review skill). No check, no loop.
3. **Can you afford the iterations?** A loop re-prompts itself until done; that costs tokens.
   Keep the `done when` check fast and add an `after N tries` thrash guard.
4. **Does the loop have the tools to verify itself?** It needs a way to implement *and* check
   its own work — the predicate command or the review skill must actually be runnable.

**Interview the user before writing it.** Walk the five decisions, asking the
high-leverage questions and offering defaults for the rest: (1) the **goal**;
(2) the **`done when`** check (test / command / scan finds-nothing / human);
(3) **`look at`** context; (4) the **action policy** (what's risky enough to gate);
(5) **stopping** (reflect on failure + an `after N tries` guard). Then the
**human gates** and the **git strategy** (default: branch + commit when done,
never push to `main`; ask if they want a PR or a worktree). Offer the defaults
inline so a confident user can accept everything at once.

### Start from a template when one fits

This repo ships a library of **best-practice starter loops** in [`templates/`](./templates/)
(see [`templates/README.md`](./templates/README.md)). When the user's request matches one,
**reach for it first** — copy it, fill in its `# TODO` lines (test commands, paths), and
adapt — instead of authoring from a blank file. They cover the everyday jobs —
`bugfix.loop`, `feature.loop`, `load-spec.loop` (deliver an existing backlog), and more;
the index is [`templates/README.md`](./templates/README.md).

Each is heavily commented and verified to parse. Still interview the user for the specifics
(goal, the real `done when`, what to gate) — the template is the skeleton, not the answer.

## Vocabulary (the whole language)

```
loop "<name>":            a self-correcting loop
pipeline "<name>":        a sequence of stages (an epic)
stage "<name>":           one stage of a pipeline (its body is a loop; a story)
flow "<name>":            a chain of loop files (each step runs a whole .loop file)
  run "<file>":           first step — runs the file; its text result is passed forward
  then run "<file>":      subsequent step — receives the previous result as context
    a human approves first  (optional per-step human gate before the step runs)
  with the result of <name>  (reference a named step's output instead of auto-carry)
  for each <var> in "<file>":  iterate items from a .yaml or .md file; run the template once per item
    run "<template>":     template receives the item text as context; fail → ask continue/stop

goal: <text>              what "done" means, in plain language
done when <predicate>     how the loop verifies itself (see Predicates)
look at: <files>, and the last failure   context the agent reads before acting
allow edits automatically, but ask me before <classes>   action policy
each cycle: plan, then act, then observe   the repeated steps (any subset, in order)
also: <pass>, <pass>      extra finishing passes run after the goal is met
use skills: <a>, <b>      named skills the loop may invoke during plan/act
remember in "<file.md>"   cross-run memory: read lessons on start, append an outcome on stop
reflect                   turn a failure into context for the next plan (the back-edge)

when it passes and the goal is met: stop
when it fails: reflect on <focus>, then plan again
when blocked: ask a human
after <N> tries: stop and warn "<message>"

a human approves the plan first        (human authors/approves the plan before acting)
a human reviews before stopping        (human judges the result before the loop stops)
a human approves before <action>       (a blocking gate before a stage, e.g. deploy)

plan from "<file>"        (read the plan from a file you control instead of generating it)

use the <method> method   schedule: <when>   runner: <agent>   target: <dir>   (config tier)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tickets-forge-dev/loop-lang](https://github.com/tickets-forge-dev/loop-lang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
