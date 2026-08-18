---
trigger: always_on
description: This repo does NOT use pull requests. Ever. NEVER run `gh pr create` (or the `gh api .../pulls` POST
---

# No pull requests — land on local `main`

This repo does NOT use pull requests. Ever. NEVER run `gh pr create` (or the `gh api .../pulls` POST
equivalent, or the GitHub UI) — not for a typo, not for a big feature, no size threshold, no
exception. This binds EVERY agent that touches this repo: the top-level worker and every sub-agent it
dispatches. If you find yourself about to open a PR, STOP — in this repo that is always the wrong move.

Land finished work on this machine's local `main` instead: work directly on `main`, or do
isolated/messy work in a git worktree on a local branch (`git worktree add <dir> -b <slug>`) and, when
it's done and you're confident, merge it back yourself (`git switch main && git merge <slug>`) and
remove the worktree. Getting the change onto local `main` is your job — never push a branch to stage a
review and never hand back an unmerged branch. Reading GitHub (issues, PRs, CI) is fine; creating or
pushing a PR is not. (`FRIZZ.md` states this in full; it is repeated here because Codex re-reads
`AGENTS.md` fresh every session and sub-agents load it, whereas the frizz worker contract can be frozen
at session start.)

Git hygiene does NOT matter here — the only thing that matters is getting your changes onto local
`main`. Many agents (and the human) work in parallel against the same repo constantly, so `main` moves
under you and the shared working tree is often dirty with someone else's in-progress edits. Do not let
that block you and do not wait it out: commit whatever is in the working tree first (an intermediate
`git add -A && git commit -m "wip: snapshot in-flight work"` is fine — committing another agent's
uncommitted changes to unblock your merge is EXPECTED, not a violation), then merge your branch in and
resolve conflicts favoring a correct build of your own change. A messy history, a WIP commit that mixes
several agents' work, an ugly merge — all fine. Never `git stash` in the shared tree (it corrupts
concurrent agents), never force-discard someone's committed work, and never stall a landing on "the
tree isn't clean." Keep merging into `main`.

## Committing out of a tree other agents are editing

**Default to `git commit -m "…" -- <paths>`.** The pathspec form commits the working-tree content of
exactly those paths on top of `HEAD` through a temp index git seeds for you, so a concurrent `git add`
by another agent cannot ride along and the shared index is left as you found it.

**Do NOT reach for `GIT_INDEX_FILE=/tmp/idx git add … && git commit`.** A temp index path that does not
exist yet starts EMPTY, not as a copy of `HEAD` — `git commit` then writes a tree holding only the
paths you added, recording *every other tracked file* as deleted. The working tree is untouched, so
nothing looks wrong until someone checks that commit out. This has cost real recovery cycles more than
once. If you genuinely need a private index (only to `git apply --cached` one hunk out of a file
another agent is also editing), run `git read-tree HEAD` immediately after setting the path.

Either way, **verify the tree right after committing**: `git ls-tree -r --name-only HEAD | wc -l`
should be roughly what it was before, not collapsed to the size of your change.

`scripts/githooks/pre-commit` backstops this — it refuses any commit that records files as deleted
while they still exist on disk, which a genuine `git rm` cannot trigger because that removes the file
from disk too. It is wired through `core.hooksPath`, which is LOCAL config, so in a fresh clone run:

```sh
git config core.hooksPath scripts/githooks
```

**Keep that value RELATIVE, and re-check it whenever the checkout moves.** Git accepts an absolute `core.hooksPath`, and this machine had one — `/Users/colinmcd94/Documents/projects/fray/scripts/githooks`, left from before the directory was renamed to `.../frizz`. Git does not warn when `core.hooksPath` names a directory that does not exist; it simply runs no hooks. Measured 2026-08-11 with a control pair: pointed at a real dir, a failing hook blocks the commit (exit 1); pointed at a missing dir, the identical commit succeeds (exit 0). So the backstop above was silently OFF for every agent commit between the rename and 2026-08-11 — the one guard against a tree-collapsing commit, gone exactly while nobody could tell. `git config core.hooksPath` must print `scripts/githooks`; if it prints an absolute path, reset it.

# Web UI completion rule — proportionate, not reflexive

Browser QA is for the changes where a browser is what actually settles the question, not for every diff
that happens to touch a `.tsx` file. Judge the change in front of you.

**Drive it in Chrome, and carry the evidence in your handoff** (inspected screenshots, console/page-error
check, the optical-review result, explicit browser-cleanup confirmation) when the change is: a new or
restructured surface, layout or interaction flow; anything judged OPTICALLY — spacing, alignment, colour,
truncation, a glyph beside text; behaviour you cannot predict from the code alone (live state, timing,
streaming, restart/recovery, a seam between processes); a fix no test pins where the bug actually lives;
or anything large, cross-cutting, or that you are unsure of.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [colinhacks/frizz](https://github.com/colinhacks/frizz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
