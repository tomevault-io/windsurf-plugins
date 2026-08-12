---
trigger: always_on
description: provides the terminal stamp — either directly on a clean initial pass, or
---

# ClaudeBoost

Research gated development for Claude Code. Every code edit is researched before
it happens, and search runs over your own indexed projects, not a scraped
knowledge base. Works standalone or with Gas Town.

## The research gate (this is the operative rule)

Every edit to a code file is checked against whether `research-agent` has run
this turn and declared that it covered that file, and nudges toward research
when it hasn't. The gate used to actually block the edit until it did; that
per turn scoping (research reset by every single message, not just a real new
task) turned out to be too disruptive in practice, so the block is gone and
the nudge plus an honest audit trail replaced it. The nudge is a PreToolUse
hook that keys off a real agent completion, not a claim of one, so the record
it checks can't be satisfied by claiming you researched, even though it no
longer refuses the edit either way.

When the gate nudges toward research:

1. **Spawn `research-agent`** (Sonnet). Tell it what you're changing, why, and the
   code you intend to write. It covers depth and breadth, checks whether the thing
   already exists, reads the project's import graph, and reports with sources and a
   `COVERS:` line naming the files it covers. That scope is what the audit trail
   checks; nothing refuses the edit, but an uncovered file shows up as uncovered.
   Wait for it before editing anyway; that's still the point. Spawn it in the
   foreground (`run_in_background: false`), never backgrounded — a backgrounded
   completion arrives later as a `TaskNotificationMessage`, not a tool result, so
   the hook that stamps the turn record never fires for it and the record never
   shows the coverage no matter how long you wait.
   Its report also names a `MATCH_STRATEGY:`. If it's `clone-and-patch`, copy the
   verbatim quoted reference as the literal starting point and make only the
   smallest set of changes that fixes the actual issue — no rewrite, no restyle,
   no swapped libraries or approaches, no added structure the reference didn't
   have. That's a hard ceiling on the diff, not a suggestion. `pattern-only`
   allows a real diff; `clone-and-patch` does not. There is no `adapt` tier.
2. There is no cheap triage tier anymore. The old one decided whether a change
   needed research WITHOUT reading the code, and that blind guess was wrong often
   enough to remove. research-agent looks first, so its judgment is grounded. It
   does real research every time it runs; do not build a triviality shortcut into
   it or any other agent.
3. Genuinely trivial work that needs no research is the human's call, not a
   model's. Run `/ps` for a quick turn that skips the gate (and the verifier) when
   you already know the change is trivial.

Markdown and non code files are exempt. So are `workspace/`, `state/`, `plans/`,
`docs/`, `.claude/`, and temp dirs.

**Depth versus breadth**, the split both agents use:
- **Depth** is the general engineering question, the one an unrelated project
  would get the same answer to. Structure, separation of responsibility,
  testability, the standard approach to this class of problem.
- **Breadth** is the task specific question. How this exact kind of thing gets
  built, what people get wrong with it, what good looks like. "What's the best
  way to build this" is breadth too, not just pitfalls.

research-agent cannot write files, and its Bash is caged to the local clean-rag
server. It reads untrusted web content, so removing its ability to act is the real
defense against a prompt injection.

## Verify by running, not by reviewing (the post write half)

Research before the edit lowers the odds of a bug. It does not confirm the code
you wrote is correct. To actually know, after writing any non trivial logic:

- **Leave one small runnable check and RUN it.** An assert, a tiny test, or
  drive the real flow. If it fails, feed the actual error back and fix once.
  Execution feedback is the highest quality per token signal there is (measured
  12 to 46 percent first try correctness gains), and it costs interpreter time,
  not tokens, except for the rare fix.
- **Do not self review your own diff in the same context.** Measured evidence
  says intrinsic self critique without external grounding is close to useless
  and sometimes makes things worse. Running the code is grounded. Re reading it
  is not.
- **A real separate context review runs on every real code change**, unless the
  human marked the turn `/ps`. It used to be reserved for high stakes surfaces
  (auth, money, SQL, a subprocess, concurrency); now it's the default after any
  code change, because green tests and correct code are different questions
  everywhere, not only there. Spawn `bad-cop` first, a fresh context critic, NOT
  the research agent (that one reads untrusted web and stays capability stripped,
  and any agent that wrote or researched the change inherits its own blind spot on
  review). bad-cop writes adversarial tests, runs the code, adds logging, and
  reports the real failures it finds, with actual execution output attached.
  If it finds nothing real, it stamps `VERIFIED:` itself: no separate
  `good-cop` run needed to re-confirm a clean adversarial pass. Only when it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MikahNiehaus/ClaudeBoost](https://github.com/MikahNiehaus/ClaudeBoost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
