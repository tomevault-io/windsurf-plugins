---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

- **Build the project**: `make clean && make`
- **Clean build artifacts**: `make clean`
- **Build for release**: `make BUILD=RELEASE`
- **Disable GUI**: `make HEADLESS=TRUE`

The compiled binary is output to `bin/robomage`.

## Testing guidelines
-Non fatal errors are not acceptable
-Draws are not acceptable
-train.py --diag and train.py --watch-scripted are helpful for checking new builds; 
--when testing with diag and --watch-scripted, supply --deck and --opponent arguments to test cards/decks relevant to recently implemented features

### Test harness for card behavior verification

`train/test_harness.py` runs the engine with `--machine --narrative --no-shuffle` so that deck file order = draw order (first 7 cards become the starting hand), and game narrative is visible alongside decoded binary state.

**Engine flags used by the harness:**
- `--no-shuffle` — skip initial library shuffle; cards are drawn in deck file order
- `--narrative` — enable game_log output in machine mode (perfect information)

**Quick start — specify hands inline:**
```bash
train/.venv/bin/python train/test_harness.py \
  --hand-a "Mountain,Lightning Bolt" \
  --library-a "Island,Island,Mountain,Mountain,Mountain,Mountain,Mountain,Mountain" \
  --hand-b "Forest,Grizzly Bears" \
  --library-b "Forest,Forest,Forest,Forest,Forest,Forest,Forest,Forest" \
  --scripted --max-decisions 30
```

**Using pre-made deck files** (for precise library sizes without auto-padding):
Write `.dk` files to `bin/resources/decks/temp/` with `1 CardName` per line (hand cards first), then pass the deck name relative to `decks/`:
```bash
train/.venv/bin/python train/test_harness.py \
  --deck-a temp/my_test_a --deck-b temp/my_test_b --scripted
```

**Play modes:**
- `--scripted` — rule-based agent (from env.py) plays both sides automatically
- `--actions "9,0,7,0,8"` — pre-scripted action index sequence
- `--interactive` — prompt for each decision (useful for step-by-step debugging)
- Default (no flag) — auto-passes every decision

**JSON scenario files:**
```bash
train/.venv/bin/python train/test_harness.py --scenario scenario.json
```
```json
{
  "name": "bolt_kills_bear",
  "hand_a": ["Mountain", "Lightning Bolt"],
  "library_a": ["Island", "Island", "Mountain", "Mountain", "Mountain", "Mountain", "Mountain", "Mountain"],
  "hand_b": ["Forest", "Grizzly Bears"],
  "library_b": ["Forest", "Forest", "Forest", "Forest", "Forest", "Forest", "Forest", "Forest"],
  "actions": [9, 0, 7, 0, 8],
  "seed": 1
}
```

**Output format** — at each decision point the harness prints:
- Narrative lines from the engine (casts, damage, zone changes, combat)
- Decoded game state (life, mana, hand contents, battlefield permanents with P/T/status, stack, graveyards)
- Available actions with human-readable descriptions (e.g. "Cast Lightning Bolt", "Target Grizzly Bears (opp)")

**Notes:**
- `--hand-a`/`--hand-b` auto-pads the library to a minimum 15-card deck. For precise small libraries (e.g. testing Thassa's Oracle with near-empty deck), write deck files manually to `decks/temp/` and use `--deck-a`/`--deck-b` instead.
- Card names in deck files omit apostrophes: `Thassas Oracle`, `Lions Eye Diamond`
- The scripted agent always keeps (no mulligan), casts spells when affordable, plays lands, and attacks with all creatures
- Temp deck files in `decks/temp/` are cleaned up automatically when using `--hand-a`/`--hand-b`; manually created files in `decks/temp/` are not

## Code Style

- Don't put new functions in main.cpp
- Don't edit my comments for spelling or punctuation. Only change them if something substantive changed.
- Avoid inline logic for anything that will be repeated; write new functions that are reusable
- Declare local functions as private in the class, if the header contains a single class/struct, if header does not contain a class, write them as static functions in global namespace C-style.
- Iterate through mEntities when possible (working within a system class), rather than iterating through all entities
- Try to consolidate iterations through entities within a function, rather than iterating through many times
- Static (local) functions should be forward declared at top of source file for clarity
- C++17 with exceptions disabled (`-fno-exceptions`)
- GUI is written in C99 with raylib
- Uses clang-format configuration in `.clang-format`

## Project Overview

Robomage is a C++ implementation of a Magic: The Gathering game engine using an Entity Component System (ECS) architecture.

Every game decision logged as an integer. Games can be replayed deterministically when provided with the correct seed.

The python side of the project enables machine learning of the game and analysis.

## Architecture

### ECS Pattern

The codebase follows an Entity Component System architecture based on [Austin Morlan's ECS tutorial](https://austinmorlan.com/posts/entity_component_system/):

- **Entities** (`src/ecs/entity.h`): Simple uint32_t IDs, maximum 5000 entities
- **Components** (`src/components/`): Pure data structs attached to entities

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ceverettkoop) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
