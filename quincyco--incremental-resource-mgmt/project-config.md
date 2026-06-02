---
trigger: always_on
description: **Project Name:** Incremental Resource Mgmt
---

# CLAUDE.md - Incremental Resource Management Game

## Project Overview

**Project Name:** Incremental Resource Mgmt
**Engine:** Godot 4.5
**Language:** GDScript
**Genre:** Incremental/Idle Game
**Current Status:** Early development with core mechanics implemented

This is an incremental game where players click on a rock/wizard to spawn particles, which can be collected for energy. Energy is used to purchase upgrades that enhance particle generation and collection efficiency.

---

## Codebase Structure

```
/
├── .git/                    # Git repository
├── .gitignore              # Godot-specific ignores
├── README.md               # Minimal project description
├── CLAUDE.md               # This file - AI assistant guide
├── project.godot           # Godot project configuration
│
├── main.tscn/.gd           # Root scene - connects all systems
├── game_manager.tscn/.gd   # Core game state and logic
├── ui_manager.tscn/.gd     # UI rendering and upgrade buttons
├── wizard.tscn/.gd         # Clickable object that spawns particles
├── particle.tscn/.gd       # Individual particle entities
├── particle_collector.tscn/.gd  # Collection area for particles
├── camera_controller.tscn/.gd   # Camera pan/zoom controls
└── icon.svg                # Godot default icon
```

---

## Architecture & Design Patterns

### Node Hierarchy

```
Main (Node2D)
├── GameManager (Node)           # Singleton-like game state
├── CameraController (Camera2D)  # Camera controls
├── UIManager (CanvasLayer)      # UI overlay
├── Wizard (Node2D)              # Click source
└── ParticleCollector (Node2D)   # Collection area
```

### Signal-Based Communication

The project uses Godot's signal system for decoupled communication:

**main.gd:10-11** - Signal connections:
```gdscript
game_manager.energy_changed.connect(ui_manager.update_energy_display)
ui_manager.upgrade_purchased.connect(game_manager.apply_upgrade)
```

**Key Signals:**
- `GameManager.energy_changed(new_energy: int)` - Emitted when energy updates
- `GameManager.particle_collected(value: int)` - Emitted on particle collection
- `GameManager.upgrade_applied(upgrade_id: String)` - Emitted after upgrade purchase
- `UIManager.upgrade_purchased(upgrade_id: String)` - Request upgrade purchase
- `Particle.particle_clicked(particle: RigidBody2D)` - Particle click event
- `Wizard.rock_clicked(click_position: Vector2)` - Rock/wizard click event

### Singleton Pattern (Godot Style)

GameManager is accessed as a pseudo-singleton via absolute node paths:
```gdscript
game_manager = get_node("/root/Main/GameManager")
```

This pattern is used in:
- particle.gd:19
- wizard.gd:13
- ui_manager.gd:13
- particle_collector.gd:12

---

## Game Mechanics

### Core Loop

1. Player clicks the Wizard (rock sprite)
2. Wizard spawns N particles (based on upgrades) with random velocities
3. Particles become RigidBody2D objects with physics
4. Player clicks particles OR they enter the collector area
5. Particles move toward the collector
6. Upon collection, energy is added to the player's total
7. Energy is spent on upgrades to enhance the loop

### Energy System

**Initial State:**
- Starting energy: 0
- Particles per click: 5
- Particle value: 1 energy each
- Collection radius: 50 units

**game_manager.gd:9-13** defines the initial game state.

### Upgrade System

**Upgrade Types** (game_manager.gd:16-21):

| Upgrade ID | Effect | Base Cost | Cost Scaling |
|-----------|--------|-----------|--------------|
| `more_particles` | +2 particles per click | 50 | 1.5x per level |
| `particle_value` | +1 energy per particle | 100 | 1.5x per level |
| `collection_radius` | +25 units collection area | 75 | 1.5x per level |
| `auto_collector` | +1 automatic collector | 200 | 1.5x per level |

**Cost Formula:** `new_cost = current_cost * 1.5` (game_manager.gd:48)

### Particle Lifecycle

**particle.gd** implements the full particle lifecycle:

1. **Spawn** - Created by wizard.gd:69 with random position/velocity
2. **Physics Phase** - Falls with gravity (0.5x), affected by linear_damp (2.0)
3. **Click Detection** - Area2D detects mouse clicks (particle.gd:59-63)
4. **Collection Phase** - Moves toward collector at increasing speed (particle.gd:50-57)
5. **Collection** - Adds value to energy, plays fade animation (particle.gd:91-100)
6. **Timeout** - Disappears after 10 seconds if uncollected (particle.gd:102-108)

**Critical Fix Notes:**
- particle.gd:17 - Particles must add themselves to "particles" group in code
- wizard.gd:59 - Particle scene path uses lowercase "particle.tscn"

---

## Development Workflow

### Running the Project

```bash
# Open in Godot Editor
godot project.godot

# Run from command line (if Godot is in PATH)
godot --path . --headless  # For testing
```

### Git Workflow

**Current Branch:** `claude/claude-md-mi3saow3iof548o1-01Q4SvRNiHggmSr4AmD39J6y`
**Main Branch:** (not specified - likely `main` or `master`)

**Branch Naming:** All development branches should start with `claude/` for AI assistant work.

### Scene Editing

- Scenes (.tscn) are in text format for version control
- Each .gd script has a corresponding .tscn file
- Scripts are attached to scene root nodes
- UI is built programmatically in ui_manager.gd:21-51 (no .tscn UI elements)

---

## Key Conventions & Best Practices

### Code Style


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quincyco/Incremental_Resource_Mgmt](https://github.com/quincyco/Incremental_Resource_Mgmt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
