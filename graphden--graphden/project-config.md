---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**How Graphden is built.** Graphden is developed by a human engineer who uses AI
coding agents as an accelerator. The engineer sets the direction, owns the design
decisions, and reviews every change before it lands. This file and the `dev/wtq/`
workflow describe how that human-directed work is organized so one developer can
move fast — they are not a claim that the software writes itself. AI involvement
is deliberate and not hidden.

## Are you a pooled feature agent? (read before editing)

Parallel feature work happens in **isolated git worktrees** behind a serialized
merge queue (`dev/wtq/`). An agent's operating contract is delivered as its
launch *message* — so `/clear` and context compaction **destroy it, while this
file survives**. If you have no memory of a contract, re-orient here before you
touch a single file.

**Am I a pooled agent?** Yes if either holds:

- your working directory is under `graphden-wt/<name>/` (one worktree per agent), or
- `git branch --show-current` prints `feature/<name>` and `bb wt list` shows `<name>` in the pool.

If neither holds, you are in the **main checkout on `develop`** — you are *not*
claimed, and the first rule below still binds you.

**Full contract: [dev/wtq/AGENT.md](dev/wtq/AGENT.md)** — read it before editing. The rules most
often violated after a context loss:

| Rule | Why it matters |
|------|----------------|
| **Claim before you edit** — `bb wt claim <name> "<summary>"`, then `cd` to the printed WORKTREE and work only there | Editing the main checkout on `develop` corrupts the shared baseline every other agent branches from |
| **Stay in your worktree** — never `cd` into another agent's worktree, never edit `develop`, never touch another agent's branch | Agents change unrelated files in parallel; your view of the repo is your branch only |
| **`bb ci` is your only local *test* command; `bb wt up` is your only live *instance*** | `bb rebuild` / `bb deploy` / `bb test-integration` / `bb test-e2e` / `bb coverage` drive the SHARED stack (`graphden-executor` on :9002) and the shared image tag that `bb test-e2e` boots — from a worktree they steal the demo and make another agent's suite test your binary. They belong to the landing gate, behind its lock. `bb wt up` gives you an isolated stack (own containers, volumes, image, ports) to see your change run |
| **Finish the job yourself** — a complete, `bb lint`-green feature goes through `bb wt merge`, then `bb wt drop`, without asking | Neither step can lose work: the gate cannot advance `develop` on a red result, and `drop` refuses an unmerged branch. Asking to merge a finished feature is ceremony, and it stalls a serialized queue on a human's reply. (A full local `bb ci` before queueing is optional solo — the gate re-runs it on the merged result; go `bb ci`-green first only when `bb wt list` shows other claimed agents.) Stop and ask only when a real decision is yours and the answer changes what you build |

**Recovering the contract and your place in it** — the branch, the worktree and
the task spec all live on disk, so nothing but the *prompt* is lost with the
context:

```bash
bb wt list             # every agent: branch, drift vs develop, last gate RESULT
bb wt status           # same, plus the recent gate runs
bb wt task <name>      # the task spec you were handed
bb wt log <name>       # full transcript of your last gate run
bb wt watch <name>     # follow a running gate: 60s ticks until RESULT, then print it
bb wt bootstrap        # reprint the discussion-phase (nameless-agent) launch prompt
bb wt kickoff <name>   # reprint the launch prompt for an already-claimed agent
```

## Design Principles (MUST READ)

**Every change must improve at least one principle without violating others.**

| # | Principle | Description |
|---|-----------|-------------|
| 1 | **Correctness first** | No feature justifies bugs. Comprehensive tests required. |
| 2 | **Minimal entities** | Resist adding new entity types, fields, or edge types. Each addition increases complexity everywhere. |
| 3 | **Explicit over implicit** | Behavior must be visible in graph structure. No magic, no context-dependent semantics. |
| 4 | **DRY** | Never define the same thing twice. Use inheritance (parent-id) and result caching for reuse. |
| 5 | **Expressiveness parity** | Can do everything classical languages can. No "sorry, you can't do that." |
| 6 | **No unnecessary expressiveness** | Don't add features just because we can. |
| 7 | **Locality of changes** | Changing one node shouldn't require changes elsewhere. |
| 8 | **Incrementality** | Adding features shouldn't require rewriting existing ones. |

**Before making changes, ask:**

- Which principle does this improve?
- Does it violate any other principle?
- Is there a simpler way?

See [docs/PHILOSOPHY.md](docs/PHILOSOPHY.md) for full rationale and module mapping.

## Project Overview

Graphden is a visual functional programming environment where functions and their compositions are stored as a graph in a database.

**Key concepts:**

- **Code = Graph in DB** — functions and arguments stored as entities

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Graphden/graphden](https://github.com/Graphden/graphden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
