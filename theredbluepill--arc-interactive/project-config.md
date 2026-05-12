---
trigger: always_on
description: Agent responsible for designing and implementing ARC-AGI-3 games.
---

# ARCAGI-3 Game Designer Agent

## Role
Agent responsible for designing and implementing ARC-AGI-3 games.

## Benchmark principle

ARC-AGI-style benchmarks are about **whether an AI can solve puzzles that humans can solve** when both operate under the **same official interface**: defined actions, observation format, and stated game rules. The goal is to measure **general problem-solving**, not which system had the **most task-specific training data** or memorized solutions.

When you author games in this repo, treat **human solvability (given that shared spec)** as a design guide: prefer **well-posed** levels—where goals are clear from layout, mechanics, or fair in-observation cues—over **under-specified** puzzles that only yield to guessing, huge search, or spoilers absent from the agent’s observation.

## Workflow

### 1. Game Design Phase
**Input**: Game concept or requirements
**Output**: Game specification

**Key Questions**:
- Grid size? (8x8, 16x16, 24x24, 64x64)
- What entities? (player, targets, walls, hazards)
- What actions? (define what ACTION1-7 mean for your game)
- Win/lose conditions?

### 2. Implementation Phase
**Input**: Game specification
**Output**: Working game in `environment_files/`

**Steps**:
1. Create directory: `environment_files/{stem}/{version}/` (two-letter stem + digits, e.g. `ez01`; version folder is usually `v1` on first landing, then often an 8-char git prefix after CI — see `CONTRIBUTING.md`)
2. Implement `{stem}.py` with:
   - Sprite definitions
   - Static levels (no PCG)
   - Game class extending `ARCBaseGame`
   - Win/lose conditions
3. Test with: `arc.make` using the full `game_id` from that folder’s `metadata.json`, or locally `uv run python run_game.py --game {stem} --version auto` — add `--mode human` for **pygame** hand-play (`scripts/human_play_pygame.py`).

### 3. Documentation Phase
**Input**: Completed game
**Output**: Updated tracking files

**Steps**:
1. Add entry to `GAMES.md` with all metadata columns
2. If you discover a **reusable** pattern (not stem-specific), add a short bullet under **Lessons learned (cross-repo)** below; otherwise rely on `GAMES.md`, `{stem}.py`, and **`scripts/render_arc_game_gif.py`** (see skill **`generate-arc-game-gif`** in **`skills/`**)
3. Optional: add **`assets/{stem}.gif`** using the **generate-arc-game-gif** skill (advancing levels + 1–2 fail clips, HUD in **`RenderableUserDisplay`**)

## Established Game Patterns

Based on `environment_files/` games (vc33, ls20, ft09):

### 1. Camera Initialization
```python
Camera(x, y, width, height, background_color, padding_color, [sprite_list])
```
- Position: always `(0, 0)`
- Width/height: 16x16 or 64x64 (match your largest level)
- Last param: list containing a custom RenderableUserDisplay object (optional but recommended)
```python
BACKGROUND_COLOR = 0
PADDING_COLOR = 4

camera = Camera(0, 0, 16, 16, BACKGROUND_COLOR, PADDING_COLOR)
# With custom UI sprite:
camera = Camera(0, 0, 16, 16, BACKGROUND_COLOR, PADDING_COLOR, [self._ui])
```

### 2. RenderableUserDisplay (UI Class)

All established games use a custom UI class that extends `RenderableUserDisplay`:

```python
from arcengine import (
    ARCBaseGame,
    Camera,
    Level,
    RenderableUserDisplay,
    Sprite,
)

class GameUI(RenderableUserDisplay):
    def __init__(self, game_state: int) -> None:
        self._state = game_state
    
    def update(self, game_state: int) -> None:
        self._state = game_state
    
    def render_interface(self, frame):
        # Draw UI overlay on frame (e.g., targets remaining, timer)
        return frame


class MyGame(ARCBaseGame):
    def __init__(self) -> None:
        # Create UI object
        self._ui = GameUI(0)
        
        super().__init__(
            "mygame",
            levels,
            Camera(0, 0, 16, 16, BACKGROUND_COLOR, PADDING_COLOR, [self._ui]),
            False,
            1,
            [1, 2, 3, 4],
        )
    
    def on_set_level(self, level: Level) -> None:
        self._player = self.current_level.get_sprites_by_tag("player")[0]
        self._targets = self.current_level.get_sprites_by_tag("target")
        # Initialize UI with target count
        self._ui.update(len(self._targets))
    
    def step(self) -> None:
        # ... game logic ...
        
        # Update UI when state changes
        self._ui.update(len(self._targets))
        
        self.complete_action()
```

### 3. Game Class __init__
```python
class MyGame(ARCBaseGame):
    def __init__(self) -> None:
        # Optional: create custom sprite object
        self._custom = CustomSprite(self)
        
        super().__init__(
            "game_id",
            levels,
            Camera(...),
            False,  # debug flag
            1,      # config value
            [1, 2, 3, 4],  # available_actions: simple actions (ACTION1–4); define semantics in step()
        )
```

### 3. on_set_level()
```python
def on_set_level(self, level: Level) -> None:
    # Store sprite references by tag
    self._player = self.current_level.get_sprites_by_tag("player")[0]
    self._targets = self.current_level.get_sprites_by_tag("target")
    
    # Optional: get level configuration data
    self._difficulty = self.current_level.get_data("difficulty")
```

### 4. step()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theredbluepill/arc-interactive](https://github.com/theredbluepill/arc-interactive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
