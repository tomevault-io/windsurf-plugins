---
trigger: always_on
description: Git Context Controller (GCC) v2 — Lean agent memory backed by real git. Stores hash + intent + optional decision notes instead of verbose markdown. Auto-bridges to aiyoucli vector memory when available. Dual mode: git-backed (lean index.yaml) or standalone (markdown fallback). Triggers on /gcc commands or natural language like 'commit this progress', 'branch to try an alternative', 'merge results', 'recover context'.
---


# Git Context Controller (GCC) v2

## Overview

GCC transforms agent memory from verbose markdown into a lean, git-backed index. Instead of duplicating what git already knows, GCC stores only **hash + intent + optional decision notes** (~50 tokens vs ~500 per entry). Full context is reconstructed on demand via `git show`.

Two modes:
- **git mode**: Real commits, lean `index.yaml`, worktree isolation, aiyoucli bridge
- **standalone mode**: Markdown fallback for projects without git (v1 compatible)

## Initialization

On first use, check if `.GCC/` exists. If not, run the init script:

```bash
scripts/gcc_init.sh          # auto-detects git/standalone
scripts/gcc_init.sh --upgrade # migrate v1 → v2
```

### Git mode structure
```
.GCC/
├── index.yaml       # Single source of truth (timeline, worktrees, decisions)
├── branches/        # Branch-specific notes (only when needed)
├── worktrees/       # Worktree tracking
└── .bridge-log      # Tracks what's been synced to aiyoucli
```

### Standalone mode structure
```
.GCC/
├── index.yaml       # Timeline in standalone mode
├── main.md          # Global roadmap (v1 compat)
├── log.md           # OTA execution log (v1 compat)
└── branches/
```

## index.yaml Format

```yaml
version: 2
mode: git              # or "standalone"
created: "2026-03-28T00:00:00Z"
config:
  proactive_commits: true
  worktree_ttl: 24h
  bridge_to_aiyoucli: auto   # auto | off | manual

current_branch: main

timeline:
  - id: C001
    hash: 85c8539
    intent: "release prep"
    branch: main
    date: "2026-02-25T21:40:00Z"

  - id: C002
    hash: a3f1b22
    intent: "auth middleware rewrite"
    note: "descartamos passport.js — too opinionated for our use case"
    branch: main
    date: "2026-02-26T10:00:00Z"

worktrees:
  - name: refactor-auth
    path: ../gcc-wt-refactor-auth
    branch: refactor-auth
    created: "2026-03-28T10:00:00Z"
    ttl: 24h
    status: active

decisions: []
```

**Key principle**: `hash` is the pointer to git truth. `intent` is why. `note` is only for decisions git can't capture (rejected alternatives, trade-offs).

## Scripts

### gcc_init.sh
Initialize GCC v2. Auto-detects git repo.
```bash
scripts/gcc_init.sh              # new project
scripts/gcc_init.sh --upgrade    # migrate v1 → v2 (backs up old files)
```

### gcc_commit.sh
Execute a real git commit and append lean entry to index.
```bash
scripts/gcc_commit.sh "implement retry logic"
scripts/gcc_commit.sh "release prep" "descartamos semantic-release por overhead"
scripts/gcc_commit.sh --staged "hotfix: null check"   # only staged files
```

### gcc_context.sh
Reconstruct context from index hashes.
```bash
scripts/gcc_context.sh                    # last 5 entries with git details
scripts/gcc_context.sh --last 10          # last 10
scripts/gcc_context.sh --hash abc123      # full details for specific commit
scripts/gcc_context.sh --summary          # one-line per entry (cheap, no git calls)
scripts/gcc_context.sh --decisions        # only entries with notes
scripts/gcc_context.sh --full             # everything
```

### gcc_bridge.sh
Feed commit data to aiyoucli vector memory. Silent no-op if aiyoucli unavailable.
```bash
scripts/gcc_bridge.sh --status            # check bridge connectivity
scripts/gcc_bridge.sh --sync              # sync all unsynced entries
# Called automatically by gcc_commit.sh when bridge_to_aiyoucli: auto
```

### gcc_cleanup.sh
TTL-based worktree cleanup and index pruning.
```bash
scripts/gcc_cleanup.sh                    # interactive cleanup
scripts/gcc_cleanup.sh --dry-run          # show what would be cleaned
scripts/gcc_cleanup.sh --force            # clean without asking
scripts/gcc_cleanup.sh --prune-index 50   # keep last 50 timeline entries
```

## Commands

### COMMIT

Persist a milestone. In git mode, executes a real commit.

**Triggers**: `/gcc commit <summary>`, "commit this progress", "checkpoint"

**Procedure (git mode)**:
1. Run `scripts/gcc_commit.sh "<intent>" ["<note>"]`
2. This stages changes, commits, captures hash, appends to index.yaml
3. If `bridge_to_aiyoucli: auto`, feeds to aiyoucli memory

**Procedure (standalone mode)**:
1. Run `scripts/gcc_commit.sh "<intent>"`
2. Appends OTA entry to log.md

**Proactive behavior**: When `proactive_commits: true`, suggest a commit after:
- Completing a function, module, or coherent unit of work
- Fixing a bug and verifying the fix
- Finishing a research/exploration phase with conclusions

### BRANCH

Create an isolated workspace. In git mode, uses real git worktrees.

**Triggers**: `/gcc branch <name>`, "try a different approach", "experiment with..."

**Procedure (git mode)**:
1. `git worktree add ../gcc-wt-<name> -b <name>`
2. Register in index.yaml under `worktrees:` with TTL
3. Agent works in worktree directory

**Procedure (standalone mode)**:
1. Create `.GCC/branches/<name>/` with summary.md, log.md
2. Register in index.yaml timeline

### MERGE


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [faugustdev/git-context-controller](https://github.com/faugustdev/git-context-controller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
