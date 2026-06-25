---
trigger: always_on
description: `overleaf-git-sync` is now a working V1 Python CLI for agent-safe synchronization between a self-hosted Overleaf Community Edition project and a local Git repository.
---

# AGENTS.md

## Project: overleaf-git-sync

## Status: V1 Implemented

`overleaf-git-sync` is now a working V1 Python CLI for agent-safe synchronization between a self-hosted Overleaf Community Edition project and a local Git repository.

The tool is intentionally not a Git remote helper and not a replacement for Overleaf Server Pro Git integration. It treats Overleaf CE as a remote project state source and publishing target, while local Git remains the conflict-resolution and review layer.

Core data flow:

```text
Overleaf CE project  <-->  ol-ce-sync HTTP backend  <-->  local Git repo  <-->  agent / editor
```

The highest-priority invariant remains:

```text
Never silently overwrite newer Overleaf changes with stale local agent output.
```

---

## V1 User Workflow

Authenticate once:

```bash
ol-ce-sync auth login --host http://localhost --email you@example.com
ol-ce-sync auth status --host http://localhost
```

If password login is blocked by captcha, SSO, or local deployment policy, log in through the browser and pass a raw Cookie header:

```bash
ol-ce-sync auth login --host http://localhost --cookie 'sharelatex.sid=...'
```

Initialize a local repo against a real Overleaf project:

```bash
ol-ce-sync init --host http://localhost --project-id YOUR_PROJECT_ID --project-name my-paper
```

Normal edit loop:

```bash
ol-ce-sync pull
# agent/editor modifies .tex/.bib/.sty/.cls/etc.
ol-ce-sync status
git diff
git add -A
git commit -m "agent: revise introduction"
ol-ce-sync push --dry-run
ol-ce-sync push
```

If Git reports conflicts, resolve locally, commit, then retry:

```bash
git status
git add conflicted-file.tex
git commit -m "resolve Overleaf sync conflict"
ol-ce-sync push
```

---

## Implemented Commands

### `ol-ce-sync auth login`

Creates `.ol-sync/session.json` containing Overleaf session cookies. This file is ignored by Git and must never be committed.

Supported modes:

- `--email` plus password prompt, using Overleaf `/login`
- `--cookie`, using a raw browser Cookie header

### `ol-ce-sync auth status`

Checks the saved session against Overleaf `/user/personal_info`.

### `ol-ce-sync auth logout`

Deletes the local session file.

### `ol-ce-sync init`

Initializes sync metadata and Git state for a real Overleaf CE project.

Current behavior:

1. Requires `--project-id`; there is no fake default project.
2. Writes `.ol-sync/config.toml`.
3. Validates the saved HTTP session.
4. Downloads the remote project zip snapshot.
5. Initializes Git if needed.
6. Imports the remote snapshot into `overleaf-remote`.
7. Creates or switches back to `main`.
8. Merges `overleaf-remote` into `main`.
9. Records sync metadata under `.ol-sync/`.

`--force` is required to initialize in a non-empty non-Git directory.

### `ol-ce-sync pull`

Downloads the latest Overleaf snapshot, imports it into `overleaf-remote`, and merges it into the current working branch.

Conservative behavior:

- Acquires `.ol-sync/lock`.
- Aborts on a dirty worktree unless `--autostash` is passed.
- Uses Git merge for conflict detection.
- Stops and reports conflicted files on merge conflict.

### `ol-ce-sync push`

Applies committed local changes back to Overleaf.

Current behavior:

1. Acquires `.ol-sync/lock`.
2. Requires a clean worktree by default.
3. Performs an internal freshness pull before generating the push plan.
4. Computes changes with `git diff --name-status --find-renames`.
5. Prints the planned operations.
6. Honors `--dry-run`.
7. Applies file operations through the HTTP backend.
8. Downloads a fresh Overleaf snapshot.
9. Verifies remote normalized state against local normalized state.
10. Updates sync metadata only after verification succeeds.

### `ol-ce-sync status`

Prints local sync state:

- current branch
- clean/dirty worktree
- last synced commit
- last remote snapshot commit
- conflicted files
- local changes since the latest remote snapshot

### `ol-ce-sync verify`

Downloads the latest Overleaf snapshot and compares it with the local normalized project tree. Exits nonzero on mismatch unless `--allow-diff` is passed.

---

## Current Config Format

Repo-local config lives at:

```text
.ol-sync/config.toml
```

Example:

```toml
[project]
host = "http://localhost"
project_id = "YOUR_PROJECT_ID"
project_name = "my-paper"

[git]
main_branch = "main"
remote_branch = "overleaf-remote"
remote_name = "origin"
require_clean_worktree_before_push = true
commit_remote_snapshots = true

[backend]
type = "http"
timeout = 16
ssl_verify = true

[auth]
profile = "default"
session_file = ".ol-sync/session.json"

[sync]
lock_file = ".ol-sync/lock"
snapshot_dir = ".ol-sync/snapshots"
last_synced_file = ".ol-sync/last_synced_commit"
last_remote_snapshot_file = ".ol-sync/last_remote_snapshot_commit"
dry_run_default = false

[ignore]
patterns = [
  ".git/",
  ".ol-sync/",
  "*.aux",
  "*.bbl",
  "*.blg",
  "*.fdb_latexmk",
  "*.fls",
  "*.log",
  "*.out",
  "*.synctex.gz",
  "*.toc",
  "*.pdf"
]
```

Ignored local files:

```gitignore
.ol-sync/snapshots/
.ol-sync/tmp/
.ol-sync/session.json
.ol-sync/cookies.json
.ol-sync/*.lock
.env
.venv/
.uv-cache/
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [genggng/overleaf-git-sync](https://github.com/genggng/overleaf-git-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
