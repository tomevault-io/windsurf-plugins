---
trigger: always_on
description: System architecture, autoload singletons, and integration patterns - Comprehensive guide to the event-driven architecture
---


# System Architecture Overview

## Autoload Singleton Hierarchy

### Load Order and Dependencies
The autoloads are loaded in this specific order (defined in `project.godot`):

1. **EventBus** - Global event system (no dependencies)
2. **GameState** - Game state management (depends on EventBus)
3. **EntityFactory** - Object pooling and spawning (depends on EventBus, GameState)
4. **EnemyTemplateManager** - Enemy definitions (no dependencies)
5. **BossTemplateManager** - Boss definitions (no dependencies)
6. **StageTemplateManager** - Stage/wave definitions (depends on template managers)

**CRITICAL**: This order MUST be maintained. Do not add autoloads between these.

### EventBus - Global Event Hub

#### Purpose
Central publish-subscribe system for decoupled cross-system communication.

#### Event Categories
```gdscript
# Player Events
signal player_spawned(player: Player)
signal player_died(position: Vector2, lives_remaining: int)
signal player_bomb_used(bomb_count_remaining: int)
signal player_hit(damage: int, invulnerable_time: float)
signal player_power_changed(new_power: int, max_power: int)

# Enemy Events
signal enemy_spawned(enemy: Enemy, template_id: String)
signal enemy_defeated(enemy_id: int, position: Vector2, score_value: int)
signal enemy_hit(enemy_id: int, damage: int, remaining_health: int)

# Boss Events
signal boss_spawned(boss: Boss, boss_id: String)
signal boss_phase_changed(boss_id: String, phase_number: int, phase_name: String)
signal boss_defeated(boss_id: String, completion_time: float, no_miss: bool)
signal boss_warning(boss_id: String, warning_duration: float)

# Combat Events
signal bullet_fired(bullet: Bullet, source: Node, bullet_type: String)
signal bullet_hit(bullet: Bullet, target: Node, damage: int)
signal item_dropped(item: Item, position: Vector2, item_type: String)
signal item_collected(item_type: String, value: Variant)

# Stage Events
signal stage_started(stage_number: int, stage_name: String)
signal stage_cleared(stage_number: int, time: float, score: int)
signal wave_started(wave_number: int, enemy_count: int)
signal wave_cleared(wave_number: int, time: float)

# Visual Events
signal screen_shake_requested(intensity: float, duration: float)
signal slow_motion_requested(time_scale: float, duration: float)
signal flash_effect_requested(color: Color, duration: float)
signal background_change_requested(background_id: String, transition_time: float)

# Audio Events
signal music_change_requested(track_id: String, fade_time: float)
signal sfx_play_requested(sfx_id: String, position: Vector2, volume: float)
signal music_intensity_changed(intensity: float)  # For dynamic music layers
```

#### Usage Patterns
```gdscript
# ✅ GOOD: Emit events instead of direct calls
func _on_enemy_health_depleted() -> void:
    EventBus.enemy_defeated.emit(
        enemy_id,
        global_position,
        score_value
    )

# Subscribe to events in _ready()
func _ready() -> void:
    if not EventBus.player_died.is_connected(_on_player_died):
        EventBus.player_died.connect(_on_player_died)

# Disconnect in cleanup
func _exit_tree() -> void:
    if EventBus.player_died.is_connected(_on_player_died):
        EventBus.player_died.disconnect(_on_player_died)

# ❌ BAD: Direct cross-system calls
func _on_enemy_health_depleted() -> void:
    get_node("/root/ScoreManager").add_score(100)  # Tight coupling!
    get_node("/root/AudioManager").play_sfx("explosion")  # Fragile!
```

#### Event Naming Conventions
- Past tense for completed actions: `player_died`, `enemy_defeated`
- Present tense for requests: `screen_shake_requested`, `music_change_requested`
- Include relevant data as signal parameters (strongly typed)
- Avoid generic names like `event`, `signal`, `update`

### GameState - Centralized State Management

#### Purpose
Single source of truth for player stats, game progress, and global state.

#### State Variables
```gdscript
# Player Stats (Read via getters, write via setters that emit signals)
var lives: int = 3
var bombs: int = 3
var score: int = 0
var continues_used: int = 0
var power_level: int = 1
var max_power: int = 4

# Game Progress
var current_stage: int = 1
var current_wave: int = 0
var difficulty: Difficulty = Difficulty.NORMAL
var rank: float = 0.0  # Dynamic difficulty from 0.0 to 10.0

# Chain and Combo System
var chain_count: int = 0
var chain_timer: float = 0.0
var score_multiplier: float = 1.0

# Game Flow Flags
var is_paused: bool = false
var is_game_over: bool = false
var is_boss_active: bool = false
var player_is_alive: bool = true
```

#### Access Patterns
```gdscript
# ✅ GOOD: Read via getters
var current_lives: int = GameState.get_lives()
var player_score: int = GameState.get_score()

# ✅ GOOD: Write via setters (emits change signals)
GameState.add_score(100)  # Emits score_changed signal
GameState.consume_bomb()  # Emits bomb_used signal, updates count
GameState.lose_life()     # Emits life_lost signal, may trigger game over

# ✅ GOOD: Query flags
if GameState.is_boss_active:
    _adjust_spawning_logic()

# ❌ BAD: Direct state duplication
class_name Player extends Node2D
var lives: int = 3  # Don't duplicate GameState.lives!

# ❌ BAD: External modification without setters

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/4t0m15) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
