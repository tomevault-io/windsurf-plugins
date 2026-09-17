---
trigger: always_on
description: Guidance for Claude Code (and other agents) working in this repository. Rules
---

# CLAUDE.md

Guidance for Claude Code (and other agents) working in this repository. Rules
only: the incidents that motivated them are in git history and the PRs that
added them.

## What Pawl is

A pure Haskell rules engine for *Magic: The Gathering*, structured as a virtual
machine with two halves:

- A closed half: the comprehensive rules (turn structure, priority, the stack,
  zones, state-based actions, the layer system, combat). Finite; it can
  genuinely be finished.

- An open half: a first-order, non-recursive, statically-analyzable effect DSL,
  loaded as runtime data, never as Haskell modules. Grows forever; that's fine.

The invariant that keeps them apart: the closed half depends on a
*classification* of effects (which layer, is it a mana ability, does it
target), never on the *identity* of an effect. The rules core must never write
`case effect of DealDamage{} -> ...`. Fusing the two halves is the single
failure mode that sinks this project --- audit for it. Keywords are the
exception that proves the rule: rule 702 is part of the rulebook, so
`case keyword of Flying` is the same kind of act as casing on `Phase`.

The second invariant: the engine never makes a player's choice. Eliding a
prompt is legitimate only for indistinguishable options, and every elision
carries an issue. Where the rules leave nothing to ask, don't prompt.

Design notes live in `docs/`. Read them by section, on demand --- a whole doc
"for context" costs tens of thousands of tokens.

- What's next?
  `gh issue list`

- Architecture rationale:
  `docs/design.md`

- Rules ground truth:
  `docs/rules.txt`, grepped by rule number --- never memory, and never a rule
  number quoted in an issue or a brief; those have been wrong.

- Prior-art evidence:
  `docs/prior-art-lessons.md`

- What the milestone era established:
  `docs/progress.md` (frozen), newest first

- A landed unit's authoritative detail:
  `docs/superpowers/`

- Dispatched as a subagent?
  `docs/agents/` --- `implementing.md` if you hold the build and will open a PR,
  `researching.md` if you are read-only, `drain-loop.md` if you orchestrate.

## Workflow

`CONTRIBUTING.md` has the loop --- issue, branch, TDD, draft PR --- and applies
to agents as written. What it doesn't say:

- A fresh `git worktree` has no gitignored `cabal.project.local`, so `pedantic`
  and `-Werror` are off and a green build says nothing about CI. Copy it in
  from the primary checkout before the first build --- by absolute path, never
  by `cd`ing there. Then `script/warm-worktree.sh seed <your worktree>`, which
  clones a finished build of `origin/main` in: GHC reuses every module whose
  source is unchanged, so the first build costs only your own diff. The
  script does the `cabal.project.local` copy too.

- Working in a worktree, NEVER `cd` to the primary checkout, not even to read.
  The isolation guard redirects `git` but not `python`, `grep`, `sed` or
  `cabal`, so every file edit made there lands on whatever branch the owner
  has checked out, alongside their uncommitted changes.

- The isolation guard refuses a command whose text holds the bare word
  `source` (the tree's top directory), a heredoc beside another command, or a
  `sed` expression carrying parentheses, a pipe, a `;` or a `$` ---
  `script/mutate.sh`'s argument most of all. Grep as `grep -rn X .
  --include='*.hs'` or against a quoted deeper path, run a heredoc as a command
  of its own, and give `mutate.sh` its script as `@FILE` from your scratchpad.

- Every worktree shares ONE stash stack, so a stash made in yours can be popped
  by an agent in another --- losing your edits and landing theirs in your tree.
  Never stash; copy the file aside and move it back.

- Prefix every `cabal` call with `script/with-build-lock.sh`, which caps
  builds at two machine-wide (the machine has 8 GB). Never `pkill -f 'cabal
  test'` --- it reaches other agents' worktrees; kill your own PID. Never pipe
  `cabal` into `head` or anything else that closes the pipe early: it
  deadlocks on SIGPIPE while holding a lock slot. Redirect to a file and read
  the file.

- Derive against `origin/main`, not the working checkout, which drifts:
  `git fetch`, then `git show origin/main:<path>` or a worktree cut from it.
  Line numbers in issue bodies are stale; grep for identifiers.

- Self-review the branch before opening the PR, and fix the findings on the
  branch. At minimum: re-check every CR citation against `docs/rules.txt`, and
  re-read every comment the change touched.

- The PR body carries the case for merging. A line each:
  - what changed and why, with `Closes #N` --- bare text, since backticks break
    the link. Never write close, fix or resolve next to an issue number you do
    not intend to close, in any phrasing including a denial; write "related to
    #N". A keyword in any branch commit survives the squash.
  - the CR citations behind it
  - the design calls made, and the alternatives rejected
  - how it was verified: suite count before -> after, the proving test, and for
    each mutation the assertion it reddened, named.
  - whether the rules core cases on an effect's *identity* --- an explicit "no"
  - what was deferred

- Keep the prose terse --- PR bodies, issue comments, commit messages and code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tfausak/pawl](https://github.com/tfausak/pawl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
