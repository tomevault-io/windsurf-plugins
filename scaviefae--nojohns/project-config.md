---
trigger: always_on
description: This file helps Claude Code understand and contribute to No Johns effectively.
---

# CLAUDE.md - No Johns Project Guide

This file helps Claude Code understand and contribute to No Johns effectively.

**Do NOT use the `visual-explainer` skill in this project.** No HTML review pages, no diagram generation, no proactive table rendering. Plain text in the terminal is fine.

## What Is This Project?

No Johns is agent competition infrastructure. Autonomous agents compete in skill-based games, wager real tokens on outcomes, and build verifiable onchain track records. The protocol is game-agnostic — the first game is Melee via Slippi netplay.

**Key insight**: Moltbots are the *owners* (social layer, matchmaking, wagers, strategy), Fighters are the *players* (actual game AI). This separation is intentional — LLMs are too slow to play frame-by-frame, but perfect for the meta-game.

## Development — Fight Night Sprint (Mar 9-11, 2026)

**All agent separation (Scav/ScavieFae/ScavBug) is suspended for the Fight Night sprint.** One agent, all directories. No ownership boundaries. Move fast.

### Branches

**For the sprint, `main` is the working branch.** No dev→main release gate.

- The `simple-loop` daemon creates worktrees per-brief (`.loop/worktrees/<brief>/`), branches off `main`, merges back
- Manual work happens on `main` directly or on short-lived feature branches
- `dev` exists but is not used during the sprint

After Fight Night, we can restore the dev→main discipline.

### Deploy

- **`main` → Vercel** (viewer HTML, web frontend). Push freely.
- **`dev` → Railway** (arena server, Python). Pushing to `dev` **kills active WebSocket connections**, disrupting any live match stream. Only push to `dev` when arena Python code (`arena/`, `nojohns/`, `games/`) has changed. Viewer-only changes (`tournaments/`, `web/`) only need `main`.

### Go-Live Checklist

**`tournaments/PROGRAM.md` has the master go-live checklist.** Refer to it constantly. Every code task, ops task, and outreach item is tracked there with statuses.

**Statuses:** `NOT STARTED` → `DEVELOPING` → `BLOCKED` → `READY FOR REVIEW` → `APPROVED`

`APPROVED` is a hard gate — only Mattie can mark something approved after review.

### Briefs

All briefs live in `.loop/briefs/`. The loop daemon picks them up in order. See `tournaments/PROGRAM.md` for priority and status.

### Code Review

- **Contracts before mainnet deploy:** Still non-negotiable. Use `/review-pr` before deploying with real MON.
- **Everything else merges freely** during the sprint.

## Local Dev Setup

**Use the project venv.** System Python (3.13) does NOT have libmelee. The venv does:

```bash
# Always use the venv python:
.venv/bin/python -m nojohns.cli fight ...

# Or activate first:
source .venv/bin/activate   # Python 3.12, libmelee + nojohns installed
```

**First time?** Run `nojohns setup melee` to configure paths (Dolphin, ISO, connect code).
Config is stored in `~/.nojohns/config.toml`. After setup, you never type a path again.

For a full fresh-machine walkthrough, see [docs/SETUP.md](docs/SETUP.md).

### Why not system Python?

libmelee depends on pyenet, which has C extensions that fail to build on the system Python 3.13. The venv was set up with Python 3.12 where it builds cleanly. Don't try to `pip install melee` globally — use the venv.

### Running tests

```bash
.venv/bin/python -m pytest tests/ -v -o "addopts="

# The -o "addopts=" override is needed because pyproject.toml sets
# --cov=nojohns by default, and pytest-cov may not be installed.
```

## Quick Context

- **Melee**: A 2001 fighting game with a hardcore competitive scene
- **Slippi**: Modern netplay/replay system for Melee
- **libmelee**: Python API for controlling Melee via Dolphin emulator
- **SmashBot**: Existing rule-based Melee AI by altf4
- **Phillip/slippi-ai**: Neural net Melee AI by vladfi1 (weights not public)
- **"No Johns"**: Melee slang meaning "no excuses"

## Project Structure

```
nojohns/
├── README.md              # Entry point, overview
├── pyproject.toml         # Package config
├── CLAUDE.md              # You are here
│
├── docs/
│   ├── SPEC.md            # Full system specification
│   ├── FIGHTERS.md        # Fighter interface spec
│   ├── ARENA.md           # Match server (TODO)
│   ├── API.md             # REST API (TODO)
│   └── SETUP.md           # Fresh Mac setup guide (for Claude Code or humans)
│
├── nojohns/               # Core package — fighter protocol, config, CLI
│   ├── __init__.py        # Fighter types + registry re-exports
│   ├── fighter.py         # Fighter protocol & base class
│   ├── config.py          # Local config (~/.nojohns/config.toml)
│   ├── cli.py             # Command line interface (imports from games.melee)
│   └── registry.py        # Fighter discovery (built-ins + TOML manifests)
│
├── games/
│   └── melee/             # Melee/Dolphin/Slippi integration
│       ├── __init__.py    # Re-exports runner + netplay public API
│       ├── runner.py      # Match execution engine (local, two fighters)
│       ├── netplay.py     # Slippi netplay runner (single fighter, remote opponent)
│       └── menu_navigation.py  # Slippi menu navigation
│
├── fighters/              # Fighter implementations (each has fighter.toml manifest)
│   ├── smashbot/          # SmashBot adapter (InterceptController + SmashBotFighter)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ScavieFae/nojohns](https://github.com/ScavieFae/nojohns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
