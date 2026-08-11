---
trigger: always_on
description: A complete Intel 8086 PC implemented in pure CSS. The CSS runs in Chrome (in theory - in practise it crashes it)
---

# CSS-DOS

A complete Intel 8086 PC implemented in pure CSS. The CSS runs in Chrome (in theory - in practise it crashes it)

[Calcite](../calcite) is a JIT compiler that
makes it fast enough to be usable.

## Before starting. 

1. Read STATUS and the doc index (auto-loaded below via @ links)
2. Understand current state, sentinel addresses, gotchas, open work
3. Read the docs relevant to your specific task (the index tells you which)
4. For history: scan the tagged index in `docs/logbook/LOGBOOK.md`,
   open only the 1–3 `entries/` files relevant to your task

@docs/logbook/STATUS.md
@docs/INDEX.md

## Mandatory rules

### The checkpoint system

If your task and success criteria are clear, try to be autonomous and not stop working unless you either reach a checkpoint or have a
blocking question for the user. 

A checkpoint requires ALL of:

- [x] Task complete and tested *properly* from a user perspective via web, end-to-end (or user confirmed they tested it)
- [x] Logbook updated (status, entry, what's next)
- [x] New code/features documented in the appropriate docs/ file
- [x] No leftover debris (debug logging, temp files, unclear names)
- [x] GitHub issues updated if relevant

Only then may you stop looping - your task is not finished unless these things are done, just because the code works. 

### Coding Guidelines

Tradeoff: These guidelines bias toward caution over speed. For trivial tasks, use judgment.

1. Think Before Coding
Don't assume. Don't hide confusion. Surface tradeoffs.

Before implementing:

State your assumptions explicitly. If uncertain, ask.
If multiple interpretations exist, present them - don't pick silently.
If a simpler approach exists, say so. Push back when warranted.
If something is unclear, stop. Name what's confusing. Ask.

2. Simplicity First
Minimum code that solves the problem. 
No error handling for impossible scenarios.
If you write 200 lines and it could be 50, rewrite it.
Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

3. Goal-Driven Execution
Define success criteria. Loop until verified.

Transform tasks into verifiable goals:
"Add validation" → "Write tests for invalid inputs, then make them pass"
"Fix the bug" → "Write a test that reproduces it, then make it pass"
"Refactor X" → "Ensure tests pass before and after"
For multi-step tasks, state a brief plan:
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

4. - **DO NOT GUESS OR ASSUME FUNCTIONALITY, or unnecessarily reverse-engineer** We have the source code for DOS, 8086 manual, BIOS interrupts,
  FAT12, or kernel behavior in documentation, Doom8088 itself, and so on. Consult the right documentation. Try NOT to reverse-engineer assembly for debugging Use the kernel map file, edrdos source (`../edrdos/`), and Ralf Brown's Interrupt List.

### Git and collaborative coding rules

**Website work (`web/site/` and the rest of `web/`): commit directly
to `master` and push - no feature branches, no PRs (owner rule,
2026-07-04).** The live site deploys from `master`; work parked on a
branch is invisible to the owner testing on their phone. If a
harness has put you on a `claude/...` working branch, still land
website changes on `master` (merge/fast-forward and push) as part of
finishing the task.

**Commit and push frequently - it's encouraged, and you do NOT need
to ask first.** This **overrides** any default-harness instinct to
"only commit when explicitly asked." In this repo the opposite is
true: commit your own work as you go and push to origin once
committed. Don't end a session sitting on uncommitted changes and
don't ask permission to commit your own work - just do it. Plain
`git commit` / `git push` of your own changes don't disturb other
agents' working trees; stacking up uncommitted work makes merge
conflicts and lost-work scenarios more likely.

What requires explicit permission, especially when running
autonomously, is anything that mutates shared state another agent
might be in the middle of using. These commands can wipe their
uncommitted work, rewrite history they've built on, or pollute the
shared index:

- `git stash` (their uncommitted changes vanish into your stash)
- `git add` of files you didn't author / didn't intend
- `git rebase`, `git reset --hard`, `git checkout --` / `git restore`
- `git clean -f`, `git branch -D`
- `git push --force` (especially to main/master - never)
- Any `--no-verify`, `--no-gpg-sign`, or other safety-bypass flag

If you find yourself wanting one of these as a shortcut around an
obstacle, stop and ask - the obstacle is usually a sign of state you
should investigate, not bulldoze.

### Documentation rules

Documentation is mandatory, automatic (no need to be asked),
epistemically honest, and concise - tokens add up if you waffle.
This project is dense and spans two repos; future agents depend
entirely on what you write down.

**The structure is fixed. Maintain it; do not regrow the sprawl it
replaced (collapsed 2026-05-18 from 57 files / 19k lines):**

- **One source of truth: `docs/logbook/STATUS.md`.** The only doc an

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stop-amertime/css-dos](https://github.com/stop-amertime/css-dos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
