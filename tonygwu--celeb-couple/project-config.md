---
trigger: always_on
description: Several agents work this repository at the same time, one agent per checkout.
---

# Notes for coding agents working in this repo

Several agents work this repository at the same time, one agent per checkout.
Git is the only channel between them. Read this file before your first git
command.

## Checkout layout on this machine

`~/Code/misc/celebrity-couple/` is a **container**, not a checkout. It holds
several checkouts of this same repository:

| Checkout | Role |
|---|---|
| `repo-0` | Working checkout for agent sessions. Edit here. |
| `repo-1` | Working checkout for agent sessions. Edit here. |
| `repo-2` | Working checkout for agent sessions. Edit here. |
| `repo-3` | Working checkout for agent sessions. Edit here. |

There is no deploy or install checkout yet. If this project ever gains one,
add a `repo-prod` checkout, give it the deploy-only role, and record that role
in this table in the same commit. A role that lives only in an agent's memory
does not exist.

Your checkout name is your **handle**. Use it when you claim work.

## The rules

1. **Sync before you touch.** `git fetch && git rebase` before you read or
   edit. Your local HEAD is a hypothesis about the repo, not a fact. A session
   that resumes after idle time re-fetches before its *first* git command, not
   before its first push.

2. **Push every completed unit, immediately.** Never end a turn with a
   publishable change sitting unpushed. If something blocks the push, say what
   it is. A held commit and a hung agent look identical from outside.

3. **Stage by name, never by sweep.** `git add -A` and `git add .` are
   forbidden here. Untracked files you did not create are another agent's work
   in progress. A tree dirty with foreign files is normal, not a mess to tidy.

   `git add -u` is the obvious thing to reach for once `-A` is ruled out, and
   it is only *safer*, not safe: it skips untracked files but still sweeps
   every TRACKED file that has been modified, including ones another agent is
   part-way through. Naming the files costs a few seconds and cannot take
   somebody else's work with it.

4. **Claim before you work.** Push the claim before you start. Add
   `[claimed by: repo-N]` to the item's line, commit that file by name, push.
   If the claim push is rejected, another agent won the item. Fetch and pick
   another.

5. **Divergence is a blocker.** More than 5 commits ahead, or any commits
   behind for more than a day, gets surfaced to the operator. Never let it
   accumulate quietly.

6. **Committed code must run from any checkout, any machine, any
   environment.** Never write a checkout-absolute path into committed code.
   Derive the repo root at runtime: `git rev-parse --show-toplevel`, or
   `Path(__file__).resolve()` in Python. A hardcoded path is correct in one
   checkout and wrong in three.

   Two other flavours of the same defect, both found in this repository:

   - **A home-absolute path.** All six quota-spending scripts defaulted
     `--account` to `/Users/tonygwu/.claude-e`, which exists on one machine
     and, on the night it was found, had 0% quota left. Use
     `packages/llmkit/accounts.py`, which refuses rather than guessing.
   - **A path that exists here and not in CI.** Three tests spawned
     `repo/.venv/bin/python`. CI installs with setup-python and has no
     `.venv`, so all three would have failed on its first run — and CI has
     never run, so nobody found out. Use `sys.executable`.

   The test for the first is that a fresh clone on another machine works. For
   the second it is `python3 -m venv` plus plain `pip install -r
   requirements.txt`, which is what the workflow does.

7. **A refusing hook is another agent talking to you.** Read its text and the
   sentinel file it names. Never `--no-verify`, never delete the hook.

8. **History rewrites: stop, fetch, reconcile.** Unknown SHAs or a
   non-fast-forward rejection mean the remote history may have moved under
   you. Stop, fetch, and reconcile against the coordinator notice at the top
   of this file. Never force-push over `main`.

9. **One session, one terminal.** Never resume the same session id from two
   checkouts at once. Concurrent appends corrupt the transcript.

## Staging by name is not enough: check the index

`git add <names>` stages those names. **`git commit` then commits the WHOLE
INDEX**, including anything another agent staged and had not yet committed. In a
shared clone that is a real and easy mistake, and rule 3 above does not prevent
it.

It happened on 2026-09-15. A commit meant to carry two files carried eight: 502
lines of `expand_roster.py`, 270 of its tests, `modules/records/wikidata.py` and
three more, all another agent's in-flight work sitting staged in the index.

**Treat a populated index in this clone as somebody else's work in progress.**
Before every commit:

```sh
git diff --cached --name-only        # is anything here not yours?
git commit -- path/one path/two      # commits ONLY these paths, index or not
```

`git commit -- <paths>` is the safe form: it ignores the rest of the index
entirely, so a foreign staged file cannot ride along.

The same incident showed a second thing worth knowing: because the commit took
the index, it took the INDEX's copy of a file that was also dirty in the working

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tonygwu/celeb-couple](https://github.com/tonygwu/celeb-couple) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
