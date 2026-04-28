---
trigger: always_on
description: - I am an experienced programmer but brand new to Godot. I have no prior knowledge of Godot-specific concepts, APIs, functions, node types, signals, or documentation.
---

# CLAUDE.md

## Developer Context

- I am an experienced programmer but brand new to Godot. I have no prior knowledge of Godot-specific concepts, APIs, functions, node types, signals, or documentation.
- When explaining Godot concepts, provide clear explanations rather than assuming familiarity.
- The developer is rebuilding this project from scratch to learn Godot hands-on. Provide guidance and explain approaches rather than writing large blocks of code unless asked.

## Guidelines

- When I propose a feature or approach, validate it against Godot best practices and game industry conventions before implementing. If my suggestion conflicts with established patterns, flag it and explain the recommended alternative.
- Prefer idiomatic Godot solutions (e.g., using signals over polling, scene composition over deep inheritance, built-in nodes over custom reimplementations).
- When making modifications, make as many edits to the .tscn file as possible before relying on .gd for functionality.

## Game Description

This is an incremental/idle Plinko game. The player operates an ever-growing Plinko machine. Coins are dropped from the top and land in slots at the bottom. Each slot grants a different reward. As the player progresses, the Plinko machine expands with more pegs, slots, and reward types.

### Art Style

This is a minimalist 3D game. Use simple primitive shapes (spheres, cylinders, boxes) — no complex meshes or high-poly models. Keep visuals clean and lightweight.

### Core Physics Approach

Do NOT use a real physics engine for coin movement. The game needs to scale to tens of thousands of coins, so all coin paths are simulated/predetermined. The pegs are purely visual. This keeps performance stable regardless of coin volume.

Coins should calculate their path **row by row**, not all at once. This way if the board changes mid-drop (e.g., rows added), the coin dynamically adapts to the new layout and always lands in the correct bucket position. The coin picks left/right randomly at each row, queries the board for the next waypoint, and determines its final bucket value at landing time.

### Key Godot Patterns to Follow

- **Signals up, calls down.** Children emit signals to notify parents. Parents call methods on children to command them. Never the reverse.
- **Autoloads (singletons)** for managers that need global access: CurrencyManager, LevelManager, SaveManager. Register these in Project > Project Settings > Autoload.
- **Resources for data.** Upgrade definitions (cost, cap, scaling formula) should be Resource subclasses or data dictionaries, not hardcoded if/else chains. One generic `buy(upgrade_id)` function replaces many individual upgrade functions.
- **Each node manages its own children.** The board builds its own pegs/buckets. The UI builds its own buttons. The drop manager owns its own timers.
- **Scenes are self-contained.** Each `.tscn` + `.gd` pair handles its own initialization, state, and cleanup.

### System Responsibilities

> **Living documentation.** This section is the authoritative map of how systems own state, emit signals, and call into each other. It is kept in sync with the code — each time a feature branch is ready to merge to `main`, the relevant entries below are updated to reflect the new behavior, signals, data flows, and cross-system relations. New systems get new entries. Removed systems are deleted. The goal is that reading this section alone is enough to understand how the systems fit together without diving into the code.

#### Project layout

- `autoloads/` — singleton managers. One subdirectory per autoload.
- `entities/` — scenes (`.tscn` + `.gd` pairs). Each is self-contained.
- `scripts/` — shared data classes, utilities (enums, reward/tier data, format utils, offline earnings).
- `style_lab/` — `VisualTheme` resource, presets under `style_lab/presets/*.tres`, plus the in-editor style lab scene.
- `assets/` — icons, sounds, fonts.

Autoload init order is set in `project.godot` and matters: `TierRegistry → CurrencyManager → UpgradeManager → LevelManager → PrestigeManager → SaveManager → SceneManager → ChallengeManager → ThemeProvider → ModeManager → ChallengeProgressManager → AudioManager`. Later autoloads may subscribe to earlier ones in `_ready`.

#### Autoloads

**TierRegistry** — `autoloads/tier_registry/tier_registry.gd`

- Pure data lookup over the ordered tier chain (gold, orange, red, ...). No mutable state.
- Consumed by nearly every manager for per-board currency, drop costs, tier indices.
- Methods: `get_tier(board_type)`, `get_tier_index`, `get_next_tier`, `primary_currency`, `raw_currency`, `get_drop_costs`.
- Emits nothing.

**CurrencyManager** — `autoloads/currency_manager/currency_manager.gd`

- Owns balances + caps for all currencies.
- Methods: `add`, `spend`, `can_afford`, `get_balance`, `get_cap`, `buy_cap_raise`, `reset`, `serialize/deserialize`.
- Emits: `currency_changed(type, new_balance, new_cap)` on every mutation.
- LevelManager, UpgradeManager (for cap-raise unlocks), and ChallengeTracker listen to `currency_changed`.

**UpgradeManager** — `autoloads/upgrade_manager/upgrade_manager.gd`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kjbrown-sandbox) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-15 -->
