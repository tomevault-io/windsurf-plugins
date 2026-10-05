---
trigger: always_on
description: Since v0.0.1, all repository changes go through focused branches and pull
---

# Agent Instructions

## Git workflow

Since v0.0.1, all repository changes go through focused branches and pull
requests. Do not commit or push directly to `main`.

1. Start from current `github/main`. If the working tree contains unrelated
   changes, preserve them and use a worktree or switch branches only when doing
   so will not disturb them.
2. Claim or create the Bead, then create a descriptively named branch such as
   `fix/<bead>-<topic>`, `feat/<bead>-<topic>`, or `docs/<topic>`.
3. Keep the branch limited to one coherent change. Do not include unrelated
   scratch files, Beads interaction logs, build output, or another session's
   edits.
4. Commit validated increments on the branch with imperative subjects and the
   required `Co-Authored-By` trailer. Push the branch and open a PR whose title,
   description, validation, and Bead references describe the final patch.
5. Before merge, rebase onto current `main`, rerun the affected checks, inspect
   the final diff, and resolve every review conversation.
6. Use GitHub's **squash and merge** operation. Delete the merged branch, update
   local `main` with `git pull --ff-only github main`, then close and sync the
   Bead with the merge commit and validation evidence.

GitHub Actions may write its dedicated `gh-pages` publication branch. Release
tags and release assets follow the release procedure; they do not permit a
direct push to `main`. Until the comprehensive CI bead (`bliss-hn1cc`) is
complete, run and report the relevant checks without making known-unreliable
jobs required merge gates.

## Session Startup

At the start of each session, orient yourself in the spec before doing work.
A `SessionStart` hook (`scripts/spec-orient.sh`) injects a banner with the
current build stage as a reminder — the read itself is still your job:

1. Read `spec/INDEX.md` (chapter/requirement map), `spec/conventions.md`
   (notation, staging rules), and `spec/stages.json` (note `current_stage` —
   only MUST requirements at or below it are gated).
2. Pull individual `spec/NN-*.md` chapters on demand for the subsystem you are
   touching. Do **not** read all ~29 spec files up front.

### Enabling the orientation hook (per-agent, one-time)

`scripts/spec-orient.sh` is committed, but the hook that runs it lives in
`.claude/settings.json`, which is **gitignored** — so each agent/checkout must
wire it up locally. Add a `SessionStart` command hook that runs the script.
For Claude Code, `.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          { "type": "command", "command": "bd prime --hook-json" },
          { "type": "command", "command": "bash scripts/spec-orient.sh" }
        ]
      }
    ]
  }
}
```

(The `bd prime` entry is the existing Beads hook; keep it and add the
`spec-orient.sh` line beside it.) Other agent runners can invoke
`bash scripts/spec-orient.sh` from their equivalent session-start mechanism, or
just run it manually at the start of a session. The script's stdout is the
orientation banner and is safe to run anytime.

## The `/grind` skill

`.agents/skills/grind/SKILL.md` (checked in) is the project's autonomous work
loop: survey the Beads queue, pick the highest-value next task (correctness +
progress toward the HotSpot-like tiered-JIT goal, without rat-holing), reprioritize
the queue, and execute it end-to-end with the GC-safety and validation discipline
below. Codex and other agents that read `.agents/skills/` pick it up directly.

Claude Code discovers skills from `.claude/skills/`, which is **gitignored**, so
each checkout wires it locally once (like the orientation hook above):

```bash
ln -sfn ../../.agents/skills/grind .claude/skills/grind   # from the repo root
```

Then invoke it with `/grind`. Editing `.agents/skills/grind/SKILL.md` updates the
skill for everyone (the symlink points at the tracked file).

## Architecture Principles

- **The interpreter MUST NOT duplicate functionality that belongs in the
  standard library.** `crates/egcl/src/cli.rs` (the tree-walking evaluator)
  should delegate to `crates/egcl-stdlib` for library behaviour — streams,
  sequences, format, conditions, pathnames, hash tables, etc. — rather than
  reimplementing it inline. Duplicated implementations drift apart and create
  incompatible data representations (e.g. the negative-fixnum stream hack in
  cli.rs vs. the heap-object streams in `egcl-stdlib::streams`). When a
  builtin needs library behaviour, wire it to the stdlib API; if the stdlib
  lacks it, add it there and call it from the interpreter.
- Before adding a builtin to cli.rs, check whether `egcl-stdlib` already
  implements it. Prefer extending stdlib over growing cli.rs.

## Common Lisp library ports

Keep EGCL compatibility changes in GitHub forks under `atgreen`, using
`#+egcl` / `#-egcl` or ASDF `:if-feature :egcl` as appropriate. Test scenarios
must import these through ocicl's `git+URL@SHA` support and commit the pinned
`ocicl.csv`; do not use edited download directories as the durable port source.
See [docs/library-forks.md](docs/library-forks.md) and `~/git/ocicl/README.md`.
The forks are the only copy: there is no `ports/` directory of patch sources.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atgreen/evergreen](https://github.com/atgreen/evergreen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
