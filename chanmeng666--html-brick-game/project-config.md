---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Starlight Breaker is a pure vanilla JavaScript HTML5 Canvas brick-breaking game with artistic design, particle effects, and Web Audio API integration. The project is designed as both an educational resource and a playable game, with comprehensive GEO (Generative Engine Optimization) implementation for AI discoverability.

**Key Architecture Principle**: Zero dependencies - everything runs natively in modern browsers using pure HTML5, CSS3, and ES6+ JavaScript.

## Technology Stack

- **HTML5 Canvas**: 2D rendering and game loop
- **Vanilla JavaScript (ES6+)**: All game logic without frameworks
- **CSS3**: Animations, gradients, and responsive design
- **Web Audio API**: Procedural sound generation
- **SVG Graphics**: Vector-based game assets (star ball, deer paddle)

## Core Game Architecture

### Game Loop Structure

The game uses `requestAnimationFrame` for smooth 60 FPS rendering:

1. **Clear Canvas** → **Update State** → **Collision Detection** → **Render Objects** → **Loop**
2. Game states: `menu`, `playing`, `paused`
3. Animation frame ID stored in `animationId` for proper cleanup

### Critical Game Objects (game.js)

- **Ball**: Position (x, y), velocity (dx, dy), radius
- **Paddle**: Position (paddleX), dimensions (paddleWidth, paddleHeight)
- **Bricks**: 2D array `bricks[column][row]` with status (1=active, 0=destroyed)
- **Particles**: `tailParticles` array for visual effects (trail and explosion types)

### Rendering Order (Important!)

Drawing sequence in `draw()` function:
1. `drawBricks()` - Background layer
2. `drawTailParticles()` - Particle effects
3. `drawBall()` - Ball with glow effect
4. `drawPaddle()` - Paddle with scaled deer SVG
5. `drawScore()` and `drawLives()` - UI overlays

### Collision Detection Optimization

The `collisionDetection()` function uses spatial partitioning:
- Pre-calculates ball boundaries
- Only checks bricks in potential collision range
- Uses column/row indexing to avoid full array iteration
- Critical: Always use early return after first collision to prevent multiple bounces

### Input Handling

**Mouse Control**: `mouseMoveHandler(e)` updates `paddleX` based on `clientX`
**Touch Control**: `touchMoveHandler(e)` and `touchStartHandler(e)` with `preventDefault()` to stop page scrolling
**Keyboard**: ESC key toggles pause/resume

## Menu System Architecture

### Menu State Management

All menus use CSS class toggling:
- `.menu` elements are hidden by default
- `.menu.active` displays the menu (`display: flex !important`)
- `#gameMenu` controls in-game UI visibility

### Key Functions

- `showMenu(menuId)`: Hides all menus, shows specified menu
- `hideAllMenus()`: Hides all menus, shows game UI
- `pauseGame()`, `resumeGame()`, `restartGame()`, `backToMainMenu()`

### Important: Menu Initialization

The game has multiple initialization points to ensure proper menu display:
1. Immediate IIFE at top of game.js
2. DOMContentLoaded event listener
3. Window load event (backup)
4. Debug function: `window.forceShowStartMenu()`

## Audio System (Web Audio API)

### Sound Generation

`playSound(frequency, duration, type, volume)` creates procedural sounds:
- **Brick Hit**: 800Hz → 600Hz square wave
- **Paddle Hit**: 200Hz sawtooth wave
- **Victory**: Ascending melody (523Hz → 1047Hz)
- **Game Over**: Descending tones (400Hz → 200Hz)
- **Background Music**: Looping 8-note melody

All sounds use exponential ramp for natural fade-out.

## Particle System

### Particle Types

1. **Trail Particles**: Follow ball movement, blue color
2. **Explosion Particles**: Generated on collisions, red/white color

### Particle Properties

```javascript
{
    x, y: position,
    dx, dy: velocity,
    radius: size,
    alpha: opacity (1.0 → 0.0),
    type: 'trail' | 'explosion'
}
```

### Performance Optimization

- Max 100 particles enforced in `drawTailParticles()`
- Alpha threshold 0.05 for removal
- Uses pre-set colors instead of gradient creation each frame
- Particles removed when radius < 1px

## High Score System

- Stored in `localStorage` as `starlightBreakerHighScores`
- Top 10 scores only
- Saved on game over and victory
- Functions: `saveHighScore(score)`, `displayHighScores()`

## GEO (Generative Engine Optimization) Implementation

### Analytics System (index.html)

The `geoAnalytics` object tracks:
- AI platform referrals (ChatGPT, Claude, Bard, Copilot, Perplexity)
- Session data (start time, referrer, user agent)
- Events: page load, game start, contact intent, session end

### AI Detection Methods

1. Document referrer analysis
2. UTM parameter checking (`utm_source=ai_recommendation`)
3. Custom tracking parameters

### Smart Link Enhancement

`smartLinkGenerator` automatically adds UTM parameters to external links for cross-platform attribution.

## Development Commands

### Local Development

```bash
# Serve with Python
python -m http.server 8000

# Serve with Node.js
npx live-server

# Or simply open index.html in browser (no server required)
```

### Testing

No automated tests currently. Manual testing workflow:
1. Open in multiple browsers (Chrome, Firefox, Safari, Edge)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ChanMeng666) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
