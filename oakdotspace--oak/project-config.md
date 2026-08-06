---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

Oak is **version control at the speed of agents**. This repo *is* its
open-source core: the reusable VCS library plus the `oak` CLI agents drive.
So you're an agent working on the agent-native VCS itself.

## This repo uses Oak, not Git

This project is version-controlled with [Oak](https://oak.space) — **not
Git**. Do not run `git` commands; use `oak` instead.

```bash
oak status          # show changed files
oak diff            # show changes vs HEAD
oak diff --branch   # show the whole branch's diff (commits + dirty files) vs its fork point on main
oak diff <branch>   # checkout-free branch diff (contribution view; --mode tree|contribution|net-merge, --against <base>)
oak diff <rev> <rev>          # diff two branches/commits (unique hash prefixes ok; paths after `--`)
oak diff <branch> --json      # per-file summary JSON; add --hunks for patches, --max-bytes <n> to bound them
oak diff --exit-code          # exit 1 when differences exist, 0 when none (like git)
oak commit          # local checkpoint only (no message — no -m; never pushes)
oak commit --push   # checkpoint, then publish the branch
oak commit <paths>  # checkpoint only the changes under the given files/directories
oak log             # show commit history
oak push            # publish the current branch to the remote
oak pull            # bring the current branch up to date

oak switch -c                      # create a generated branch from latest available main (keeps dirty files)
oak switch -c my-feature           # create a named branch from latest available main (keeps dirty files)
oak switch -c --clean              # create a clean generated branch, discarding dirty files
oak desc "what this branch does"   # set the current branch's description
oak switch                         # pick a branch interactively
oak switch my-feature              # switch to an existing branch (fetched from the remote when not local)
oak merge                          # merge the current branch into main (CI-gated server-side)
oak merge --wait                   # wait for CI on the branch head to conclude, then merge (up to 30 min; --wait N for N minutes)

oak ci status                      # CI state for the current branch head — what the merge gate checks
                                   # (exit 0 success, 1 failure/no runs, 3 still running)
oak ci runs [--limit N]            # list recent CI runs (id, branch, commit, status, duration)
oak ci logs <run-id>               # a run's step-by-step logs
oak ci rerun <run-id>              # re-dispatch a run at the same commit (infra flakes); prints the new run id
```

Branches are flat: every branch parents directly onto `main` (you can't stack
one branch on another). Commits carry no message — the **branch description**
(`oak desc`) is the narrative and becomes the squash-merge message. Your work
is isolated until you `oak push`, `oak commit --push`, or `oak merge`. `oak
switch -c` uses recently refreshed local `main` immediately; otherwise it
tries to refresh `main` and falls back to local `main` if the remote is
unavailable. Run `oak --help` for the full command reference.

> Claude Code reads `CLAUDE.md`, which is just a one-line `@AGENTS.md` import
> pointing here — so this file is the single source of truth.

## Machine-readable output contract

Oak's `--json` output is a stable surface agents may build durable habits
on. The rules:

- **Schemas are append-only within a `schema_version`.** New fields may
  appear at any time; existing fields are never removed or renamed and
  never change meaning without a `schema_version` bump. Parse leniently:
  ignore fields you don't recognize.
- **Absent means default.** Optional fields are omitted when they carry
  their default value (e.g. a missing `category` means `"source"`, a
  missing `binary_or_large` means `false`).
- **Every payload is self-describing.** `recommended_next_commands`
  contains exact invocations for the natural next step — prefer running
  one of those over guessing flags. Paged payloads carry a
  `changed_files_page` block with `next_offset` and a ready-to-run
  `next_page_command`.
- **Budgets are explicit.** Hunk-emitting output honors `--max-bytes`;
  when truncated, the payload says so (`hunks_truncated: true`, per-file
  `patch_omitted: true`) and names the command that fetches the rest.
- **Scripting without parsing:** `oak diff --exit-code` exits `1` when
  differences exist and `0` when there are none (matching `git diff
  --exit-code`), on top of the exit-code contract in `oak --help`.
  Predicted conflicts count as differences.
- **External diff tools:** set `OAK_DIFF_TOOL` to replace the interactive
  browser with your own tool over two materialized trees. The tool must
  block until you are done (the trees are temporary) — e.g.
  `OAK_DIFF_TOOL="code --wait --diff"`.

## What an Oak Space is

An **Oak space** is a directory where an agent mounts a repo once per
task. You create it with:

```bash
oak space new <org> [dir]   # e.g. oak space new acme
```

Inside that directory, **every task gets its own subdirectory**, and each
repo subdirectory is a separate `oak mount` on its own virtual branch. So a
space for the `acme` org might hold:

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oakdotspace/oak](https://github.com/oakdotspace/oak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
