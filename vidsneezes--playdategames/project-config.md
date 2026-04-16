---
trigger: always_on
description: Manages the active scene. Global instance: `GAME_WORLD`. Called from `playdate.update()`.
---

# Project Context for AI Assistants

## What Is This?

A Playdate console game built with Lua. Uses a lightweight ECS (Entity-Component-System) pattern. Game jam project for a small team (2-4 people). The architecture is designed to minimize merge conflicts by isolating work into separate files.

## Platform: Playdate

- Handheld game console by Panic
- Screen: 400 x 240 pixels, **1-bit color** (black and white only, no grayscale)
- Default frame rate: 30 FPS
- Inputs: D-pad, A button, B button, physical **Crank** (unique rotary input), Accelerometer
- Language: Lua 5.4 with custom extensions (`+=`, `-=`, `*=`, `/=`)
- Module system: `import "path/file"` (NOT `require`). No `.lua` extension in imports. All imported code shares the global scope.
- **No standard Lua libraries**: `io`, `os`, and `package` are not available. Use Playdate SDK APIs instead.
- Audio formats: WAV and AIFF only. No MP3, no OGG.
- Asset paths in code use **no file extension**: `gfx.image.new("Images/player")` not `"Images/player.png"`

## Architecture: ECS

### Entity (`source/ecs/entity.lua`)
A plain Lua table with an `id` field, an `active` flag, and component data stored as named fields.

```lua
local player = Entity.new({
    transform = Transform(200, 120),
    velocity = Velocity(0, 0),
    playerInput = PlayerInput(3),
})
-- Access: player.transform.x, player.velocity.dx
-- Destroy: player.active = false (cleaned up automatically at end of frame)
```

### Component (`source/components.lua`)
Constructor functions that return plain data tables. No methods, no behavior. All defined in a single file.

```lua
function Transform(x, y, rotation)
    return { x = x or 0, y = y or 0, rotation = rotation or 0 }
end
```

Available components: `Transform`, `Velocity`, `SpriteComp`, `PlayerInput`, `CrankInput`, `Collider`, `AudioSource`, `Health`.

### System (`source/systems/*.lua`)
Each system declares required components and receives only matching entities. One file per system.

```lua
PhysicsSystem = System.new("physics", {"transform", "velocity"}, function(entities, scene)
    for _, e in ipairs(entities) do
        e.transform.x += e.velocity.dx
        e.transform.y += e.velocity.dy
    end
end)
```

Systems: `PlayerSystem`, `CrankSystem`, `PhysicsSystem`, `CollisionSystem`, `AudioSystem`, `RenderSystem`.

**Execution order** (set in `game_scene.lua:onEnter()`): input systems first, logic systems second, render system always last.

**Inter-system communication**: Systems never call each other. They communicate by reading/writing component data. Example: `CollisionSystem` sets `entity.audioSource.shouldPlay = true`, then `AudioSystem` reads it and plays the sound.

### Scene (`source/scenes/*.lua`)
A scene owns entities and systems. Has `onEnter()`/`onExit()` lifecycle hooks.

- **GameScene**: Uses the full ECS loop (default `Scene:update()` runs all systems)
- **MenuScene / GameOverScene**: Override `update()` for direct drawing without ECS

Scene transitions: `GAME_WORLD:queueScene(GameOverScene())` — safe to call from anywhere, happens at start of next frame.

### World (`source/ecs/world.lua`)
Manages the active scene. Global instance: `GAME_WORLD`. Called from `playdate.update()`.

## File Structure

```
source/
├── main.lua                -- DO NOT MODIFY. Entrypoint: imports + boot.
├── ecs/                    -- DO NOT MODIFY. Core ECS engine.
│   ├── entity.lua
│   ├── system.lua
│   ├── scene.lua
│   └── world.lua
├── components/             -- One file per domain. Add to the matching domain file.
│   ├── core.lua            -- Transform, Velocity, Health
│   ├── input.lua           -- PlayerInput, CrankInput
│   ├── visual.lua          -- SpriteComp
│   ├── collision.lua       -- Collider
│   └── audio.lua           -- AudioSource
├── systems/                -- One system per file. Each has SDK docs in comments.
│   ├── player_system.lua
│   ├── crank_system.lua
│   ├── physics_system.lua
│   ├── collision_system.lua
│   ├── audio_system.lua
│   └── render_system.lua
├── scenes/                 -- Scene constructor functions.
│   ├── menu_scene.lua
│   ├── game_scene.lua
│   └── gameover_scene.lua
├── lib/utils.lua           -- clamp, lerp, distance, random_float, setBackground, loadConfig
├── Images/                 -- PNG image assets
├── Sounds/                 -- WAV/AIFF audio assets
└── Data.json               -- Game config (loaded via loadConfig)
```

## Code Generation Guidelines

When writing code for this project:

1. **Follow existing patterns.** Look at existing system/scene files before creating new ones.
2. **New systems**: Create in `systems/`, use `System.new(name, requiredComponents, updateFn)`. Include Playdate SDK reference comments at the top. Then add `import` line in `main.lua` and register in scene's `onEnter()`.
3. **New components**: Add constructor function at the bottom of the matching file under `components/` (e.g., collision-related → `components/collision.lua`). Create a new domain file if needed and add its `import` to `main.lua`.
4. **New scenes**: Create in `scenes/`, return a `Scene.new()` instance from a constructor function. Add `import` line in `main.lua`.
5. **Never use `require`** — Playdate uses `import`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Vidsneezes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
