---
trigger: always_on
description: handles all cube branching automatically.
---

# bgsage — Open Sage Bot Engine Library

Neural-network backgammon engine with C++ core and Python interface. Licensed under MPL-2.0.

## Repo Boundary Rule

**This is a standalone library repo (`bgsage/`). When the working directory is
pointed at this folder, ALL new files, edits, and commits MUST stay within this
repo.** Never create or modify files outside this repo, even if parent CLAUDE.md
files (from a host project that vendors this repo as a submodule) are loaded as
context. New Python modules go in `python/bgsage/`, new scripts in `scripts/`,
new C++ in `cpp/`.

The same applies to *runtime* paths: every directory the code references —
compiled extension, weights, data, output dirs — must resolve inside `bgsage/`.
Scripts must not reach into a parent directory for `bgbot_cpp.pyd`, model
weights, or output logs. The compiled `bgbot_cpp.pyd` belongs in `bgsage/build/`,
default output directories resolve under `bgsage/logs/`, and scripts should set
`_PROJECT_ROOT = _SCRIPT_DIR.parent` (the bgsage repo root) — never
`.parent.parent` (which points at the host project). Several existing scripts
in `scripts/` still resolve paths via the host project root; treat that as
legacy and fix it when convenient.

## Git Worktree Rules

**CRITICAL: When working in a git worktree, ALL file operations (reads, edits,
writes, new files, builds, script execution) MUST use the worktree path — never
the main repo path.** The worktree path is shown in the environment as "Worktree
path" and is the primary working directory for the session.

- The worktree has its own branch. Commit and push from the worktree, then merge
  to main via PR or local merge — do NOT commit directly to main.
- Use relative paths or the worktree path for all tool calls. If you see
  yourself using the main repo path (e.g. `<repo-root>/` instead of
  `<repo-root>/.claude/worktrees/<name>/`), STOP and fix it.
- New files created in the main repo path will NOT be on the worktree branch.
- The MSVC build directory (`build_msvc/`) is shared across worktrees. After
  building, copy the `.pyd` to the worktree's `build/` directory.

## Project Structure

```
cpp/                         # C++ core (all performance-critical code)
  include/bgbot/            # Public headers
  src/                      # Implementation (.cpp/.cu)
  pybind/bindings.cpp       # pybind11 Python bindings
  CMakeLists.txt            # Full build (CUDA + CPU)
  CMakeLists_cpu.txt        # CPU-only build (Docker / macOS)
python/bgsage/               # Python package
  analyzer.py               # Public API: checker + cube analysis
  types.py                  # Dataclasses (Probabilities, MoveAnalysis, etc.)
  board.py                  # Board utilities (flip, possible_moves, etc.)
  weights.py                # Production model registry, WeightConfig, model discovery
  data.py                   # .bm file loading, training data parsing
  gnubg.py                  # GNUbg CLI wrapper for reference evaluation
  matchinfo.py              # Match play take points, dead cube take points, gammon prices
  text_export.py            # Galaxy/XG-compatible text export + move notation
  xg_compare.py             # Parse XG .xg files; compute per-game PR stats vs XG
scripts/                     # Training & benchmarking scripts
tests/                       # Python tests
models/                      # Production weights (5 files per model stage)
data/                        # GNUbg benchmark + training data + bearoff DB
```

## Architecture

**C++ core with Python presentation layer.** All compute (move generation, NN
evaluation, game simulation, training, benchmarking) runs in C++. Python handles
orchestration, configuration, data loading, and results display.

### 5-NN Game Plan Strategy

5 separate neural networks, selected by game plan classification:

| Network    | Hidden | Inputs | Description |
|------------|--------|--------|-------------|
| PureRace   | 200    | 196    | Contact broken, `is_race()` true |
| Racing     | 400    | 244    | Racing game plan, contact exists |
| Attacking  | 400    | 244    | Blitzing/hitting strategy |
| Priming    | 400    | 244    | Building a prime |
| Anchoring  | 400    | 244    | Defensive anchor strategy |

**Topology**: N_inputs -> N_hidden (sigmoid) -> 5 outputs (sigmoid)

**Outputs**: P(win), P(gammon_win), P(backgammon_win), P(gammon_loss), P(backgammon_loss)
- Equity = 2*P(win) - 1 + P(gw) - P(gl) + P(bw) - P(bl)

### Board Representation

- `std::array<int, 26>` — indices 1-24 are points (positive=P1, negative=P2)
- Index 0: P2 bar (always >= 0). Index 25: P1 bar (always >= 0)
- Board is ALWAYS from the perspective of the player on roll
- `flip()` reverses + negates + swaps bar after every move
- Starting: `[0,-2,0,0,0,0,5,0,3,0,0,0,-5,5,0,0,0,-3,0,-5,0,0,0,0,2,0]`

### Output Semantics (Critical)

The NN outputs probabilities from the perspective of the player who just moved,
representing the state **after the player's move, before the opponent rolls**.

To get pre-roll probabilities for the current player: (1) flip the board,
(2) evaluate the NN, (3) invert probs (P(win)->1-P(win), P(gw)<->P(gl), P(bw)<->P(bl)).

**Tempo effect**: `evaluate(board)` != `invert(evaluate(flip(board)))` — these differ
by one tempo. Being on roll is an advantage.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markbgsage/bgsage](https://github.com/markbgsage/bgsage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
