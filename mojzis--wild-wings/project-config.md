---
trigger: always_on
description: This file provides comprehensive guidance for AI-assisted development on the Wild Wings project.
---

# 🤖 CLAUDE Development Guide

This file provides comprehensive guidance for AI-assisted development on the Wild Wings project.

## 📁 Project Structure

```
wild-wings-prototype/
├── src/
│   ├── App.js                      # Root component, state routing
│   ├── components/                 # React UI components
│   │   ├── GameCanvas.jsx          # Main 60fps game loop
│   │   ├── MainMenu.jsx            # Level selection UI
│   │   ├── Settings.jsx            # Flight sensitivity controls
│   │   └── ElderEncounter.jsx      # Bird facts display modal
│   ├── game/                       # Core game logic (classes)
│   │   ├── Level.js                # Level definitions & generation
│   │   ├── Player.js               # Bird character & physics
│   │   ├── Physics.js              # Physics constants
│   │   ├── AbilitySystem.js        # Ability management
│   │   ├── Obstacle.js             # Collision objects
│   │   ├── Collectible.js          # Wind feathers
│   │   └── GameStateManager.js     # Save/load, progression
│   └── data/
│       └── birdFacts.js            # Educational content
└── public/
    └── index.html                  # Entry point
```

## 🎯 Key Architectural Principles

### 1. Separation of Concerns
- **Components (`/components`)**: React UI, rendering, user interaction
- **Game Logic (`/game`)**: Pure JavaScript classes, no React dependencies
- **Data (`/data`)**: Static content, configuration

### 2. Game Loop Architecture
- **GameCanvas.jsx** is the orchestrator (60fps loop)
- Game classes handle their own state and logic
- React state triggers re-renders only when necessary
- Canvas rendering happens outside React lifecycle

### 3. State Management
- **Local State**: React hooks for UI state
- **Game State**: Class instances in GameCanvas
- **Persistent State**: GameStateManager + localStorage
- **Global Settings**: localStorage for user preferences

## 🔑 Key Components Deep Dive

### GameCanvas.jsx (Main Game Loop)
**Location:** `src/components/GameCanvas.jsx`

**Responsibilities:**
- 60fps game loop via `requestAnimationFrame`
- Input handling (keyboard events)
- Physics updates (gravity, velocity, position)
- Collision detection (obstacles, collectibles, safe zones)
- Rendering pipeline (background → player → HUD)
- Game state transitions

**Key Methods:**
- `gameLoop()` - Main update/render cycle
- `handleInput()` - Keyboard state processing
- `render()` - Canvas drawing pipeline

**State Variables:**
- `playerRef` - Player instance
- `levelRef` - Current Level instance
- `abilitySystemRef` - Ability management
- `cameraOffsetX` - Horizontal scrolling position

### Level.js (Level Definition)
**Location:** `src/game/Level.js`

**Current Levels:**
- `Level.createLevel1()` - "First Flight" (lines 273-303)
- `Level.createLevel2()` - "Storm Chaser" (lines 305-347)

**Adding New Levels:**
```javascript
static createLevel3() {
  const config = {
    id: 3,
    name: 'Your Level Name',
    difficulty: 'medium',
    width: 3000,
    obstacles: [/* obstacle configs */],
    collectibles: [/* collectible configs */],
    safeZones: [/* safe zone configs */]
  };
  return new Level(config);
}
```

**Obstacle Generation:**
- Uses `generateRandomizedBranches()` for dynamic layouts
- Ensures minimum gaps (100px horizontal, 120px vertical)
- Avoids safe zones with 150px buffer

### Player.js (Bird Character)
**Location:** `src/game/Player.js`

**Physics Constants (from Physics.js):**
- Gravity: 0.20 (base, scales with sensitivity)
- Flap Strength: -8 (upward velocity)
- Terminal Velocity: 6
- Horizontal Speed: 3px/frame

**Key Methods:**
- `update(input, level)` - Physics calculations
- `render(ctx, cameraOffsetX)` - Bird drawing
- `getBounds()` - Collision box

**Ability Integration:**
- Player checks `abilitySystem.activeAbility`
- Modifies physics based on active ability
- Renders particle effects

### AbilitySystem.js (Ability Management)
**Location:** `src/game/AbilitySystem.js`

**Ability States:**
- `ready` - Can be activated
- `active` - Currently in effect
- `cooldown` - Recovering, cannot activate

**Adding New Abilities:**
1. Add ability config to `this.abilities` map
2. Update `birdFacts.js` with educational content
3. Implement effect in `Player.update()` or `GameCanvas.gameLoop()`

**Example:**
```javascript
abilities: {
  'your_ability': {
    id: 'your_ability',
    name: 'Ability Name',
    duration: 3000,    // ms
    cooldown: 10000,   // ms
    state: 'locked',
    lastUsed: 0
  }
}
```

### birdFacts.js (Educational Content)
**Location:** `src/data/birdFacts.js`

**Structure:**
```javascript
{
  id: number,
  species: string,
  fact: string,           // 50-100 words, Lexile 200-500L
  ability: string,        // Ability ID
  abilityName: string,
  color: string,          // Hex color
  icon: string           // Emoji
}
```

**Content Guidelines:**
- Keep facts age-appropriate (grades 1-3)
- Include specific numbers/measurements
- Connect fact to ability unlock
- Use engaging, enthusiastic tone

## 🧪 Testing Strategy

### Current Tests
**Location:** `src/components/GameCanvas.test.jsx`

**Coverage:**
- Component rendering
- Input handling
- Game state initialization

### Testing Conventions
- Use React Testing Library

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mojzis/wild-wings](https://github.com/mojzis/wild-wings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
