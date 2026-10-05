---
trigger: always_on
description: This is TA: Byte Tactics, a matching decompilation of Total Annihilation (1997):
---

# Instructions for coding agents

This is TA: Byte Tactics, a matching decompilation of Total Annihilation (1997):
C++ that compiles, with the original Visual C++ 5.0 compiler, to byte-identical
machine code. Work is handed out as GitHub issues. Each issue lists a few
functions by address; you claim one, decompile its functions on your own
branch, and open a pull request. A human-run orchestrator re-checks every
function, merges the pull request and hands out the next issues.

Read this file, then `docs/agent-guide.md` (the technical guide: tools, rules,
naming, and hundreds of solved patterns). Follow both exactly.

## 1. Check the setup

Run these from the root of your clone of the repository (the "main
checkout"). A contributor setting up for the first time follows
`CONTRIBUTING.md` first.

```sh
gh auth status                      # must be logged in to github.com
ls toolchain/msvc5-sp3/BIN/CL.EXE orig/TotalA.exe
uv run tools/check.py 0x401070      # must print MATCH
```

If any of these fail, stop and tell the human; do not try to install things.

## 2. Pick and claim an issue

Every `decomp` issue is open to every model. Issues labelled
`hard` hold the biggest functions (over 1000 bytes); the label only marks
their size, not a narrower list of models. Every function is worked until it
matches or stops improving (see below).

```sh
gh issue list --label decomp --state open --search "no:assignee" --limit 20
```

Take the lowest-numbered issue from your list, unless the human told you which
size label to work on (`size:medium`, `size:large`, `size:huge`, `near-miss`).
GitHub search lags a minute or more behind claims and label changes, so the
list can show issues that are already taken. Before claiming, check the issue
itself:

```sh
gh issue view <N> --json assignees,labels,comments \
  --jq '{assignees: [.assignees[].login], labels: [.labels[].name], claim: ([.comments[].body | select(startswith("Claimed by") or startswith("Released")) | split("\n")[0]] | last)}'
```

`claim` is the most recent claim or release comment. Skip the issue if it has
an assignee, if `claim` starts with "Claimed by", or if its labels are not for
you. An issue whose `claim` starts with "Released" (the orchestrator frees
stale claims that way) or is null is free. Then claim it:

```sh
gh issue edit <N> --add-assignee @me
gh issue comment <N> --body "Claimed by <tool> / <model> on $(hostname) at $(date -u +%H:%MZ). Branch issue-<N>."
gh issue view <N> --comments
```

If `gh issue edit --add-assignee` fails because you are not a collaborator on
the repository (outside contributors can't assign themselves), the "Claimed by"
comment alone is your claim; the orchestrator will assign you. Several agents
can share one GitHub account, so the assignee only says "taken"; the comment
says by whom. If `gh issue view` shows a "Claimed by" comment from a
different agent after the last "Released" comment and before yours, you lost
the race: comment "Lost the claim race, the earlier claim stands" (never start
it with "Released", which would free the issue), do not unassign, and go back
to the list for another issue.

## 3. Work in your own copy

```sh
cd "$(tools/worktree.sh <N>)"       # creates .worktrees/issue-<N> on branch issue-<N>
```

Several agents run at once, so never edit files in the main checkout. Do all
work, compiling and checking inside that folder. Keep scratch files in
`build/scratch/<first address of the issue>/`.

## 4. Decompile

For each function in the issue:

1. `uv run tools/ctx.py <addr>` shows the disassembly, the callees and their
   calling conventions, and Ghidra's pseudo-C.
2. Look for already-matched neighbours and near-copies under `src/`
   (grep for a distinctive offset, string or callee address) and copy them.
   The files are in folders by subsystem (`docs/tidy-up.md`);
   `uv run tools/sources.py <addr>` prints the file of any function.
3. Write the function in its file: `uv run tools/sources.py <addr>` prints it
   when there is one, and `uv run tools/modules.py <addr>` the path a new one
   takes (`src/<folder>/<module>_<address>.cpp`). A new file has
   `// Decompiled by <model>. Names are provisional.`
   as its first line, where `<model>` is the model you actually are. When you
   finish a file another model started, make it
   `// Decompiled by <their model>, finished by <model>. Names are provisional.`;
   never remove another model's credit from a first line.
4. `uv run tools/check.py <addr>` until it prints MATCH. Use
   `uv run tools/checkall.py <addr> ...` for a whole batch and
   `uv run tools/headers.py <addr>` when registers or operand order won't
   budge.
5. When a function is close (about 90% or more) and stuck, run
   `uv run tools/permute.py <addr>` (`docs/permuter.md`). It tries thousands
   of meaning-preserving rewrites of your file for 15 minutes and writes the
   best one to `build/permute/<addr>/best.cpp`, with `best.diff` beside it.
   Read the diff and tidy it before you copy anything into your file: the
   output can contain temporaries named `tmp0`, helpers named `inl0` and
   other leftovers that need a sensible name or can go. Re-check with
   `check.py` after each edit, since a change that looks dead can be needed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HectorBailey/byte-tactics](https://github.com/HectorBailey/byte-tactics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
