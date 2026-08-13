---
trigger: always_on
description: The bottleneck in AI coding is human attention. romp lets one person direct many
---

# romp — repo instructions

## Philosophy
The bottleneck in AI coding is human attention. romp lets one person direct many
agents by spending that attention where it counts and surfacing only what is
worth acting on, so they keep the focus and flow that good work needs while
running them all in parallel. Every feature should serve that aim:
- **Spend attention, don't drain it.** A feature should take load off the user's
  working memory, not add to it. Glanceable by default; mechanics one click away.
- **Make re-engagement cheap.** Speak in the user's terms, the outcome and the
  why, never the agent's play-by-play, so picking a thread back up costs a glance.
- **Interrupt only when the human is the bottleneck.** "Needs you" means a
  decision only they can make. Waiting on a peer, a build, or another session is
  not that. Every false interrupt is a broken flow state.
- **Scale to parallelism.** Features should hold up across many concurrent
  sessions and let agents coordinate among themselves, handling the details the
  user never needs to see.
- **Never lose the thread.** Context persists, dead sessions revive with their
  history, nothing important silently drops, so stepping away is safe.

## Vocabulary — do not use the word "fleet" (user rule, 2026-08-01)
"Fleet" is not the user's word and they don't like it. Don't reach for it in new
prose, code, identifiers, commit messages, UI copy, docs, or anything else you
write for them. Say the plain thing: **sessions**, "the sessions you're
running", "your sessions" — and "across every session" where you'd have written
"fleet-wide".

The word is still all over the existing repo — a UI pane named Fleet
(`ui/webview/fleet.ts`, `fleet-pane.css`, `#fleet-list`), plus docs, plans and
tests. That backlog is deliberately NOT a rename-on-sight task: a sweep like
that is its own change, and it needs the user's call on what the pane is called
instead. This rule governs what you ADD. Rephrasing a line you were already
editing is welcome; renaming identifiers is not, until that call is made.

## Privacy — no real session data or personal identifiers in this repo
This repo may go public; assume every commit is permanent and world-readable.
- **Never copy real recorded data into the repo** — no real prompts,
  transcripts, per-turn summaries, postal messages, or message ids, not even
  "just one" to reproduce a bug. When a real session triggers a bug, write a
  SYNTHETIC reproduction: invented prompt text, placeholder UUIDs
  (`11111111-2222-...`), hostname `TESTHOST`. Live data belongs only under
  `~/.local/state/romp/` and `~/.claude/` (both outside the repo).
- **No personal identifiers** in code, comments, fixtures, docs, or commit
  messages: no names, machine/host names, vault names, emails, or absolute
  home paths (use `$HOME`/`~`).
- **Paraphrase the user, never quote them.** The `(the user <date>: ...)`
  attribution convention that explains WHY code exists is fine — but it must
  paraphrase, never embed a verbatim quote of what they typed. A quoted
  utterance is real recorded data. Write `(the user 2026-07-02, who wanted one
  shared picker)`, NOT `(the user 2026-07-02: "same code path, one picker")`.
- **No real session or goal names from OTHER projects.** A bug that surfaced in
  some other session is documented with a SYNTHETIC session name (`web`, `api`,
  `TESTHOST`) and an invented goal title — never the real project's nickname or
  goal text (which leaks what that unrelated project is). Add any coined
  project/session nickname to `~/.config/romp/private-strings.txt` so the test
  catches it. Reuse the neutral demo domain the doc screenshots use (a
  `notes-api` with `web`/`api`/`tests` sessions) rather than inventing per-test
  worlds.
- Two machine-local backstops enforce this, neither a substitute for the rule:
  the `.githooks/pre-push` hook greps every PUSHED commit's tree for the strings
  in `~/.config/romp/private-strings.txt` (absent file → no-op, so contributors
  are unaffected; it scans pushed shas, not the working tree, so it arms every
  worktree — a working-tree scan missed a leak pushed from a peer worktree on
  2026-07-25); and the maintainer's clone carries an UNTRACKED
  `tests/test_no_personal_identifiers.py` that scans the working tree for the
  same strings plus that machine's hostname and home path. The pytest file is
  deliberately not in the repo: one machine's identifiers mean nothing on anyone
  else's clone, and a contributor's test run must never trip over it. Both read
  text only, so screenshots and recordings under `docs/assets/` must be
  eyeballed for on-screen session content before release.

## Worktrees — work on an isolated worktree by default (user rule, 2026-06-29)
Do ALL non-trivial work on its own git worktree, not the shared main tree — concurrent
peer sessions clobber/commit each other's uncommitted edits in the shared tree (a peer's
broad `git add` will sweep up your work). Conventions:
- **One worktree per session, named after the session.** Branch + directory take the
  session's name, e.g. session `bugsdk2` → branch `bugsdk2`, dir `../romp-bugsdk2`
  (`git worktree add -b <session> ../romp-<session> HEAD`). So a glance at
  `git worktree list` says who owns what.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [romp-on/romp](https://github.com/romp-on/romp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
