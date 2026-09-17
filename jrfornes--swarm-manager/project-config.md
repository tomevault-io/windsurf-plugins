---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A file-based orchestrator for running multiple Claude Code agents in parallel,
each in an isolated git worktree, with a static HTML dashboard. **Roadmap:**
evolve into a local **forge host** (many work-items, Attention, bounded
pipeline concurrency) — see `plan.md` Part III. Deliberately dependency-free:
Python 3 stdlib only, no framework, no database, no build step.

## Commands

Start the dashboard server (serves the UI and the JSON/log API on port 8787):

```bash
python3 server.py
```

Launch an agent job (run from the repo root; background with `&` to launch several at once):

```bash
python3 launch_agent.py <role> <repo_path> "<task description>" [--permission-mode MODE] [--timeout SECONDS] [--item NAME] [--skills-dir PATH]
```

`<role>` (`builder`/`tester`/`reviewer`/anything) is just a label shown on the dashboard — roles are not enforced. To actually constrain what an agent may do, pass `--permission-mode` (one of `default`, `acceptEdits`, `bypassPermissions`, `plan`; default `plan` for the `reviewer` role, `acceptEdits` otherwise) or add flags to the `claude` invocation in `launch_agent.py`. `--skills-dir` (default `.claude/skills`) is only recorded in `meta.json` and used by `pipeline.py` to build stage prompts.

`launch_agent.py` exits `0` **only** when the job lands in `done` — `blocked`, `failed`, and timeouts all exit `1`. Chaining, `pipeline.py`'s stage gate, and the future queue worker all depend on that; preserve it if you touch the exit path.

Run one work-item through ordered skill stages (background with `&` to run pipelines in parallel):

```bash
python3 pipeline.py <repo_path> "<work-item description>" --stages spec-author,verifier,implementer
python3 pipeline.py --target my-nx-app "<work-item description>"
python3 pipeline.py --target my-nx-app "<work-item description>" --workflow docs/reference/swarm-workflows/workflow-4-pack-smoke.json --dry-run
```

Stages default to the target repo's workflow manifest (see `docs/workspace-contract.md`). `--dry-run` prints the stage commands without launching anything. `pipeline.py` does **not** validate a manifest stage's `permission_mode`; a typo only surfaces when `launch_agent.py` rejects it at that stage.

Clean up after jobs (worktrees, `agent/*` branches, and `jobs/` entries are never auto-removed). `cleanup.py` handles all three and refuses to discard uncommitted or unmerged agent work unless `--force`:

```bash
python3 cleanup.py <job-id> [<job-id> ...]   # specific jobs
python3 cleanup.py --done                    # every done/blocked/failed job
python3 cleanup.py --all                     # every job that isn't running
python3 cleanup.py --dry-run --all           # preview, change nothing
python3 cleanup.py <job-id> --force          # discard uncommitted/unmerged work too
```

There are no tests, linter, or dependencies to install. "Verify" in this repo means a scripted smoke test: create a throwaway git repo, run a real `launch_agent.py` job (or `pipeline.py --dry-run`) against it, and check the resulting `meta.json` / dashboard state. `plan.md` records a concrete **Verify** step under every phase that has landed, plus a "Testing note" and a retained regression recipe ("Verify workspace abstraction") near the end — reuse those rather than inventing new ones.

## Layout

This repo is the orchestrator; agents never run *in* it. `<repo_path>` must be a separate, real git repo, since `launch_agent.py` runs `git worktree add` inside it. Alternatively, `pipeline.py --target NAME` resolves `repo` from a local `targets.local.json` (template: `targets.example.json`).

There is **no `.gitignore` yet**. `jobs/`, `worktrees/`, and `targets.local.json` (which holds absolute local paths) are runtime/local state and must not be committed; add a `.gitignore` before committing after a real run.

- `target_config.py` — loads `targets.local.json` and each consumer's `.swarm/profile.json` (`skills_dir`, `workflow` paths).
- `docs/workspace-contract.md` — the consumer-repo contract: registry keys, `.swarm/profile.json`, `.swarm/workflow.json`.
- `docs/reference/swarm-workflows/` — reference workflow manifests (1- through 6-stage packs) with a picker README; copy into a consumer's `.swarm/` or point `--workflow` at one.
- `jobs/`, `worktrees/` — per-job state and checkouts, created at runtime. Both are cleaned only by `cleanup.py`. `launch_agent.py` also appends a JSONL debug trace to `jobs/debug-3b000d.log`; it is best-effort, never affects status, and is skipped by `server.py` / `cleanup.py` because it is not a directory.
- `public/` — the static dir `server.py` serves. **`public/index.html` is not checked in**; `server.py` creates an empty `public/` on startup, so the dashboard is a 404 until the page is added.
- **Planned (Part III):** `work-items/` (or `forge/items/`) for forge metadata; `worker.py` for queue + max in-flight pipelines.

## Architecture

The `jobs/` directory **is** the entire state store. There is no database and no shared in-memory state between the entry points — they communicate only through files on disk.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jrfornes/swarm-manager](https://github.com/jrfornes/swarm-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
