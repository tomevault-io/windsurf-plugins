---
trigger: always_on
description: Sports EPG generator. Uses **bd (beads)** for issue tracking. Start with `bd ready`.
---

# Agent Instructions - Teamarr

## Overview

Sports EPG generator. Uses **bd (beads)** for issue tracking. Start with `bd ready`.

## CRITICAL: Database Safety

**NEVER delete `teamarr.db` or `data/teamarr.db`.** The database contains user-configured teams, templates, settings, and history that cannot be recreated. Schema changes use migrations (`INSERT OR REPLACE`, `ALTER TABLE`) - deleting the database is NEVER required and will cause data loss.

**Stack**: Python 3.11+, FastAPI, SQLite | Frontend: React + TypeScript + Vite + Tailwind

## Start of Session

1. Re-read this file and follow it exactly
2. Switch to `dev` branch: `git checkout dev && git pull`
3. Check for work: `bd ready`

If you forget this workflow after a context compaction, re-read this file before continuing.

## Local Testing

Run `./dev.sh` to start both servers in one terminal:

```bash
./dev.sh                 # fast restart — skips cache refresh
./dev.sh --update-cache  # restart with full cache refresh
```

- **Backend** (FastAPI): `http://localhost:9195` — Python venv, `app.py`
- **Frontend** (Vite HMR): `http://localhost:5173` — proxies `/api` → `:9195`

Use `:5173` during development for hot-reload. `Ctrl+C` stops both.
Re-running `./dev.sh` kills existing servers first, so it doubles as a restart.

By default the script skips the startup cache refresh for fast restarts. Pass `--update-cache` when you need fresh team/league data from providers. Cache can also be refreshed manually via the UI button.

**Always use `./dev.sh` to start or restart the dev environment.** It handles cleanup of old processes automatically.

**When to restart:**
- After making backend (Python) code changes
- If Playwright browser automation can't connect to `localhost:5173`
- After schema or configuration changes

## Quick Reference Commands

```bash
bd ready                              # Find available work
bd show <id>                          # View issue details
bd update <id> --status in_progress   # Claim work
bd close <id>                         # Complete work
bd sync                               # Sync beads data
```

## Development Workflow

**Critical:** Work from `dev` branch, not `main`.

### Development Steps

1. **Check for work**: `bd ready` or `bd list`
2. **Claim work**: `bd update <id> --status in_progress`
3. **Implement the change**
4. **Run quality gates** (MANDATORY when shipping):
   ```bash
   ruff check teamarr/
   pytest tests/ -v
   cd frontend && npm run build
   ```
5. **Close the bead**: `bd close <id>`
6. **Push to dev** (MANDATORY):
   ```bash
   git add <changed-files>
   git commit -m "Brief description"
   git push origin dev
   ```

**Critical shipping rules:**
- Work is incomplete until `git push` succeeds
- Never stop before pushing—it leaves work stranded locally
- Never say "ready to push when you are"—YOU must push

### Roadmap & Feature Planning

Use beads epics to plan larger features:

```bash
bd create "Feature name" --type epic --label roadmap
bd create "Implementation step 1" --parent <epic-id>
bd create "Implementation step 2" --parent <epic-id>
bd dep add <step2-id> <step1-id>    # step 2 blocked by step 1
```

When asked to plan a feature, create an epic with implementation beads that have proper blockers and predecessors. Use `bd list --label roadmap` to see the roadmap.

### Release Workflow (`/release`)

When the user says **"release"**, **"/release"**, or **"version bump"**, execute this workflow:

1. **Determine scope** — `git log origin/main..origin/dev --oneline` to see all commits in the release
2. **Ask version** — suggest patch (x.y.Z) vs minor (x.Y.0) based on scope. User decides.
3. **Quality gates** (MANDATORY):
   ```bash
   source .venv/bin/activate
   ruff check teamarr/
   pytest tests/ -v
   cd frontend && npm run build
   ```
4. **Version bump** — edit `pyproject.toml` line 7, commit "Bump version to x.y.z"
5. **Push dev** — `git push origin dev`
6. **Merge to main** — fast-forward merge:
   ```bash
   git checkout main && git pull origin main
   git merge dev --no-edit
   git push origin main
   git checkout dev
   ```
7. **Create GitHub release** — `gh release create v<version> --repo Pharaoh-Labs/teamarr --target main` with summarized release notes (not commit-by-commit — group into categories)
8. **Generate Discord changelog** — use the Release Template below, output ready to paste
9. **Update plans/STATUS.md** — add release to changelog, update version

**Rules:**
- Never release with failing tests or lint errors
- Release notes should be human-readable summaries, not raw commit messages
- Group related commits into single bullet points

## Changelog Format

When asked for a changelog, **always** produce Discord-ready markdown. Two templates:

### Dev Push Template

Get version from `pyproject.toml` line 7, append `-dev+<short_hash>` of HEAD commit.

```
## 🚀 v<version>-dev+<hash> — <YYYY-MM-DD>

🐛 **Bug Fixes**
- <one-liner> (#issue) (`hash`)

✨ **New Features**
- <one-liner> (#issue) (`hash`)

⚡ **Enhancements**
- <one-liner> (#issue) (`hash`)

🎨 **UI/UX**
- <one-liner> (#issue) (`hash`)

🔧 **Under the Hood**
- <one-liner> — thanks @contributor (#PR) (`hash`)
```

### Release Template

```
## 🎉 v<version> — <YYYY-MM-DD>

🐛 **Bug Fixes**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pharaoh-Labs/teamarr](https://github.com/Pharaoh-Labs/teamarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
