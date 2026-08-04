---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A fully interactive puzzle game implemented in Python using Pygame with advanced merged tile mechanics. Players select an image, configure puzzle dimensions, and solve the puzzle by dragging and dropping tiles. When tiles are correctly placed adjacent to each other, they automatically merge into larger pieces that move together. Features include automatic tile merging, smart swap validation, win detection, solution hints, and smooth drag-and-drop mechanics.

## Running the Game

```bash
# Install dependencies
pip install -r requirements.txt

# Run the game
python main.py
```

## Project Structure

```
jeu puzzle/
├── main.py                 # Entry point and game state management
├── requirements.txt        # Python dependencies (pygame)
├── images/                 # Puzzle image assets (JPEG/PNG)
├── screens/               # Screen modules
│   ├── menu_screen.py     # Image selection menu
│   ├── config_screen.py   # Tile configuration input
│   └── game_screen.py     # Main puzzle display with drag-and-drop
└── utils/                 # Utility modules
    ├── constants.py       # Game constants and configuration
    ├── image_utils.py     # Image processing functions
    └── puzzle_logic.py    # Puzzle state and mechanics
```

## Architecture

### Game Flow
1. **Menu Screen** - User selects an image from the images folder
2. **Config Screen** - User inputs X and Y tile dimensions (2-20 range)
3. **Game Screen** - Puzzle is automatically shuffled; player drags tiles to solve

### Key Components

**main.py**
- `PuzzleGame` class manages overall game state using `GameState` enum
- Main game loop handles events, updates, and rendering
- State transitions between MENU → CONFIG → GAME → MENU

**screens/menu_screen.py**
- `MenuScreen` class displays available images as clickable buttons
- Automatically discovers images from the images folder
- Supports hover effects and click selection

**screens/config_screen.py**
- `ConfigScreen` class handles user input for tile dimensions
- Input validation ensures values are within MIN_TILES to MAX_TILES range
- Supports keyboard input, Tab/Enter navigation, and mouse interaction

**screens/game_screen.py**
- `GameScreen` class displays the puzzle with drag-and-drop and merged tile mechanics
- Individual tiles and merged groups can be dragged and swapped
- Detects merged groups when initiating drag operations
- Smart swap validation prevents invalid group swaps
- Renders merged tiles without internal borders (seamless appearance)
- `_draw_merged_group_borders()` - Draws perimeter borders around merged groups
- `_draw_dragging_group()` - Renders entire merged group while dragging
- `_draw_group_border()` - Helper to draw external borders only
- Visual feedback: hover effects, drag transparency, merged group highlighting
- Win detection automatically triggers when puzzle is solved
- UI controls: Shuffle, Restart, Solution (shows correct tiles in green)
- Keyboard shortcuts: S (shuffle), R (restart), Space (toggle solution), ESC (menu)
- Image is resized to fit screen while maintaining aspect ratio

**utils/image_utils.py**
- `get_available_images()` - Scans images folder for supported formats
- `load_image()` - Loads image from disk
- `resize_image_to_fit()` - Resizes while maintaining aspect ratio
- `create_puzzle_tiles()` - Subdivides image into a 2D grid of tiles
- `calculate_tile_display_rect()` - Calculates tile position with spacing

**utils/constants.py**
- Color definitions for UI elements (including drag, hover, win states)
- Screen dimensions and FPS settings
- Tile border width, spacing, and drag transparency
- Font sizes and UI element dimensions
- Game button dimensions for in-game controls

**utils/puzzle_logic.py**
- `PuzzleState` class manages tile positions, merged groups, and game logic
- `shuffle()` - Randomly shuffles tiles by performing random swaps
- `swap_tiles()` - Swaps two individual tiles at given positions
- `swap_groups()` - Swaps entire merged groups (validates size/shape matching)
- `is_solved()` - Checks if puzzle is in solved state
- `get_tile_at_position()` - Returns which original tile is at a position
- `get_correctness_grid()` - Returns grid showing correctly placed tiles
- `_update_merged_groups()` - Detects and tracks merged tile groups
- `_find_connected_correct_tiles()` - BFS algorithm to find connected correct tiles
- `get_group_containing_position()` - Returns merged group at a position (if any)
- `can_swap_groups()` - Validates if a group swap is possible (size/shape/bounds checking)

## Game Mechanics

### Merged Tile System
- **Automatic Merging**: Adjacent correctly placed tiles automatically merge into larger groups
- **Group Movement**: Merged tiles move together as one unit when dragged
- **Smart Validation**: Swaps only allowed between matching sizes:
  - Single tile ↔ Single tile: ✓ Allowed
  - Single tile ↔ Merged group: ✗ Blocked
  - Merged group ↔ Merged group (same size/shape): ✓ Allowed
  - Merged group ↔ Merged group (different size): ✗ Blocked
- **Visual Integration**: No borders between merged tiles, only around group perimeter

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [R-luiz/tile-puzzle](https://github.com/R-luiz/tile-puzzle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
