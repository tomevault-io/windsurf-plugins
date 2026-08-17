---
trigger: always_on
description: Project-specific instructions for AI coding agents working on the papertrade repo.
---

# AGENTS.md

Project-specific instructions for AI coding agents working on the papertrade repo.
Kilo auto-loads this file. Read it before doing anything that touches git state.

## Stack

- **Frontend**: Vite + React 18 + TypeScript, recharts for charts, supabase-js, axios, zustand, react-router.
- **Backend relay**: Python 3, FastAPI (`bloomberg-service/app.py`), runs on uvicorn.
- **Edge functions**: TypeScript on Supabase (`supabase/functions/`), deployed via `supabase functions deploy`.
- **Bloomberg**: `xbbg_sapi` (private wheel) + `blpapi` (Bloomberg native), installed in `.venv-bb` Python venv. NOT in git.
- **External**: Vercel (frontend hosting), Cloudflare quick-tunnel (`cloudflared.exe` in user home), Supabase (DB + functions).

Daily startup is documented in `SETUP.md`.

## Worktrees — read this before creating one

This repo uses git worktrees heavily. They are managed by Kilo's Agent Manager, which lives in the VS Code extension. Worktrees are created under `.kilo/worktrees/<name>/` and get their own branch.

**Setup is automatic.** When Agent Manager creates a worktree, it runs `.kilo/setup-script.ps1` which:
- Junctions `.venv-bb` (Python venv with blpapi + xbbg_sapi) from the main repo
- Hard-links `xbbg_sapi-*.whl` from the main repo
- Copies `bloomberg-service/.env` from the main repo (root-level .env is auto-copied, nested .env is not)
- Junctions `frontend/node_modules` from the main repo (saves 60-180s of npm install)
- Prints per-worktree ports: uvicorn 8001-8009, vite 5174-5182

**Run is automatic.** `.kilo/run-script.ps1` starts `npm run dev` in `frontend/` on the worktree's vite port. Use the Run button in Agent Manager, or invoke directly.

### Helpers (manual use, no Agent Manager required)

From the main repo root in PowerShell:

```powershell
# List all worktrees with their branches, ahead/behind main, and changed files
powershell -ExecutionPolicy Bypass -File scripts\worktree-list.ps1

# Pick from a list and run its frontend (symlinks node_modules, instant start)
powershell -ExecutionPolicy Bypass -File scripts\preview-worktree.ps1

# Specific worktree
powershell -ExecutionPolicy Bypass -File scripts\preview-worktree.ps1 -Name <branch>

# Force a real npm install in the worktree (instead of symlink)
powershell -ExecutionPolicy Bypass -File scripts\preview-worktree.ps1 -Name <branch> -Install
```

### Rules for agents using worktrees

1. **One worktree per agent session.** Don't share a worktree between two agents.
2. **Don't edit files in `main`.** Use a worktree. Editing main corrupts other agents' assumptions.
3. **Commit often, even WIP.** `git commit -am "wip: <short reason>"` is fine. Lost work hurts; noisy commits don't.
4. **Before removing a worktree, check if its branch has commits not on main.** `git log main..<branch> --oneline` shows you. If non-empty, the work is real — don't delete.
5. **Don't use `git stash` against the working tree on OneDrive.** OneDrive holds file locks; `git stash` may silently fail to capture your changes (this has caused data loss in this repo — see "Known issues" below).
6. **Conflicts between worktrees**: merge or rebase the original/base branch INTO the conflicted worktree first, resolve there, then push. Don't use `git stash` for cross-worktree state.
7. **Parallel agents should edit non-overlapping files.** Two agents editing `PortfolioDetailPage.tsx` will produce unmergeable patches. Coordinate file scope up front.

## Output discipline (applies to all agents)

- **Smallest change that works.** Don't refactor adjacent code.
- **Match existing patterns.** If the codebase uses dataclasses, use dataclasses.
- **No premature abstractions.** Three similar lines > a helper for one use case.
- **No new top-level dependencies without asking.** Check `package.json` and `requirements.txt` first.
- **No comments explaining *what*.** Only *why*. No "TODO: consider" or "future improvement" notes.
- **No error handling for cases that can't happen.** No logging/metrics/feature flags unless asked.
- **New files are a last resort.** Extend existing ones.
- **Tests are required for:** bug fixes, new public functions, changed behavior. Don't write tests for internal helpers.
- **No new docs unless asked.** No READMEs for one-off scripts.

## What NOT to do (banned unless explicitly asked)

- Don't introduce DI containers, "manager" classes, or service locators.
- Don't add config files / env vars for things that are constants.
- Don't add type stubs (`Protocol`, `ABC`) unless the codebase already uses them.
- Don't create skills/commands/agents unless they fit a repeated pattern (more than 2-3 uses).
- Don't write tests that mock what you can call directly.
- Don't run `git push` or create commits/PRs unless explicitly told to.

## Known issues / context

- **`.chroma/`, `supabase/.temp/`, root `package.json`/`package-lock.json` are noise.** All gitignored. Don't be alarmed if you see them as untracked.
- **OneDrive sync** holds file locks during git operations and can cause `git stash` to silently lose changes. Prefer committing or branching over stashing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cenweien/papertrade](https://github.com/cenweien/papertrade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
