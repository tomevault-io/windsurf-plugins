---
trigger: always_on
description: Bashcrawl is an educational text-based adventure game that teaches terminal/shell commands through immersive fantasy gameplay. Directories are game rooms, files named `scroll` are educational content, and executable scripts (`treasure`, `potion`, `spell`, etc.) are interactive encounters. The repo has two player surfaces, one harness, and one embedded framework: the pure-bash game under `entrance/` (played with a real shell), the static web trainer in `web/` (built by `scripts/export_static_web.
---

# Bashcrawl Copilot Instructions

## Project Overview

Bashcrawl is an educational text-based adventure game that teaches terminal/shell commands through immersive fantasy gameplay. Directories are game rooms, files named `scroll` are educational content, and executable scripts (`treasure`, `potion`, `spell`, etc.) are interactive encounters. The repo has two player surfaces, one harness, and one embedded framework: the pure-bash game under `entrance/` (played with a real shell), the static web trainer in `web/` (built by `scripts/export_static_web.py`), the MCP playtest harness in `src/playtest/`, and TermForge in `termforge/` — the universal terminal framework the web emulator was extracted into (kernel in `termforge/core/`, vendored to `web/assets/js/vendor/termforge/` by `make web-build`; edit the core, never the vendor mirror; node hosts serve the game over TTY/telnet). Runtime dependencies: standard POSIX shell tools. Python 3.10+ required only for the web export/validation tooling (`scripts/`) and the playtest harness; node 20+ only for the TermForge tests and hosts (`make test-js`, `make tty-demo`).

## Architecture

### Directory-as-Room Structure

Main progression path: `entrance/` → `cellar/` → `armoury/` → `chamber/`

Hidden areas unlocked by collecting treasures (all rooted at `entrance/`):
- `.chapel/` → `graveyard/`, `courtyard/aviary/hall/library/` — teaches `grep`, `find`, pipes
- `.vault/` → `stronghold/nursery/lab/` — teaches variables, env, process substitution
- `.scrap/` — **directory** (not a file) containing a scroll that teaches `ln -s`
- `.rift/` → `arena/pit/`, `spire/mezzanine/` — advanced scripting, checksums
- `entrance/workshop/` — player-created room (does not exist until player runs `mkdir`)

Each room teaches 1-3 related terminal concepts with progressive difficulty.

### Key Components

| Component | Purpose |
|-----------|---------|
| `setup.sh` | Permissions setup, system checks, makes game files executable. Sources `lib/colors.sh` |
| `help.sh` | Root-level shim delegating to `src/help/bashcrawl_help.sh`. Context-aware: detects player location, tracks progress patterns, suggests next commands |
| `src/help/` | `bashcrawl_help.sh`, `ai_engine.sh`, `command_suggester.sh`, `init_help.sh`. YAML data in `src/help/data/` |
| `lib/` | `colors.sh` (color constants), `log.sh` (JSONL session logging), `yaml_reader.sh` (YAML parsing helpers), `reset.sh` (game-state reset) |
| `entrance/.functions` | Defines `gameover()` (combat death) and `help()` (delegates to `$BASHCRAWL_ROOT/help.sh`) |
| `web/` | Static web trainer (GitHub Pages bundle). Data built from the YAML registries by `scripts/export_static_web.py` (`make web-build`); preview with `make web-preview` |
| `src/playtest/` | Lean MCP playtest harness: `python3 -m playtest.mcp_server` with `PYTHONPATH=src`; sessions scored by `python3 -m playtest.scorer` |

### Game Content Files
- **`scroll`** — Plain-text educational content (NOT a directory). Format varies by depth; see `.github/instructions/scrolls.instructions.md`
- **`treasure`** — Bash scripts: add inventory items, unlock hidden rooms via `mv`
- **`potion`** — Interactive y/n prompts teaching `read`, `export`. Checks `${HP:-0} -gt 0` (not just `-n "$HP"`)
- **`spell`** — Teaches `ln -s`; creates symlink portals between areas
- **`statue`** — Combat teaching `let`/arithmetic. Uses `.statue_defeated` flag — does NOT modify tracked files
- **`ghost`, `monster`** — Enemy encounters in hidden areas
- **`goblet`** — In `.vault/stronghold/`, checks for `orb1` in inventory, unlocks `.rift`

### Game Mechanics

**Inventory** — comma-separated env var:
```bash
export I=amulet,$I           # Add item
grep --quiet amulet <<< "$I" # Check for item
```

**Room unlocking** — rename hidden dirs (may be 2+ levels up):
```bash
mv ../../.chapel ../../chapel 2>/dev/null
```

**Health** — numeric env var:
```bash
export HP=15      # Set by potions
let "HP=HP-5"     # Combat damage
```

**Combat flags** — touch files, never destructive:
```bash
touch .statue_defeated   # Checked on re-entry to skip encounter
```

**All game executables** follow this structure:
1. `#!/usr/bin/env bash` shebang
2. 14-line "wandered out of bounds" boilerplate comment
3. Game state checks (`grep` inventory, test `$HP`)
4. Story output via `cat << EOF` heredocs (plain text, no ANSI colors)
5. Instruct player to run `export` commands (never mutate git-tracked files)
6. Unlock hidden rooms via `mv`

## Build and Test

```bash
# Setup
bash setup.sh              # Make game files executable, validate system

# Play
cd entrance && cat scroll  # Play the game — the filesystem IS the game
make web-preview           # Serve the static web trainer at http://127.0.0.1:8000

# Help
bash help.sh               # Contextual help
bash help.sh commands      # Command reference

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bamr87/bashcrawl](https://github.com/bamr87/bashcrawl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
