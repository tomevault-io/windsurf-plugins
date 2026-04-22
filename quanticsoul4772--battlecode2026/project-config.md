---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Context

**Battlecode 2026** - MIT competitive programming competition where AI bots control rat armies to gather cheese, damage cats, and eliminate enemy kings. Games run under strict bytecode constraints (17,500/turn for baby rats, 20,000/turn for kings).

**Current Status**: Active development, Tournament runs Jan 12 - Jan 31, 2026. Multiple bot iterations exist (ratbot through ratbot6), with ratbot5 being the current champion (~3,500 lines, sophisticated kiting and combat).

## Build Commands

All commands run from `scaffold/` directory:

```bash
cd scaffold

# Development
./gradlew build                    # Compile all bots
./gradlew test                     # Run JUnit tests
./gradlew run                      # Run match (uses gradle.properties settings)

# Custom matches
./gradlew run -PteamA=ratbot5 -PteamB=examplefuncsplayer -Pmaps=DefaultMedium

# Code quality
./gradlew spotlessApply            # Auto-format with Google Java Format
./gradlew checkstyleMain           # Lint main code (currently disabled in build.gradle)

# Submission
./gradlew zipForSubmit             # Create submission.zip for tournament upload

# Utilities
./gradlew listPlayers              # Show all bot implementations
./gradlew listMaps                 # Show available maps
./gradlew update                   # Update to latest Battlecode engine
```

**Match Configuration**: Edit `scaffold/gradle.properties` to set teamA, teamB, maps, and debug options.

**Viewing Replays**: Run client from `scaffold/client/Battlecode Client.exe`, load `.bc26` files from `scaffold/matches/`.

## Critical Architecture Patterns

### Bot Iterations & Evolution

**Bot progression** (oldest to newest):
- `ratbot` - Original modular architecture with separated algorithms/ and logging/ packages
- `ratbot2` - Combat/Economy role separation experiment
- `ratbot3` - Simplified single-file approach
- `ratbot4` - Population-based strategies (12 rats, 6 attackers / 6 collectors)
- **`ratbot5`** - Current champion: sophisticated kiting state machine, enemy tracking, overkill prevention, adaptive strategies (~3,500 lines)
- `ratbot6` - In development: Counter-strategy to beat ratbot5 using value functions and mobile king

**Current active bot**: `ratbot5` (configured in gradle.properties)

### Code Organization Philosophy

**Two approaches exist in this codebase:**

1. **Modular (ratbot)**: Separated packages for algorithms, logging, with RobotPlayer delegating to BabyRat/RatKing classes
   - Easier to test and reuse
   - More files to navigate

2. **Monolithic (ratbot3/4/5)**: Single RobotPlayer.java with all logic
   - Faster bytecode (no method overhead)
   - Easier to optimize holistically
   - Current competitive approach

**Use monolithic style for new competitive bots** - bytecode optimization is critical in competition.

### Bytecode Optimization is Critical

**Battlecode is bytecode-limited**, not time-limited. Every operation costs bytecode:
- Sense operations: ~100-500 bytecode
- Movement/actions: ~200-400 bytecode
- Math.abs(), Math.sqrt(): ~50 bytecode each
- Division: ~30 bytecode
- Object allocation: EXPENSIVE (avoid in main loop)

**Optimization patterns used in ratbot5:**
```java
// GOOD: Inline abs instead of Math.abs()
int dx = rx - currentLoc.x;
if (dx < 0) dx = -dx;

// GOOD: Bitshift instead of division
int rallyX = (me.x + cachedMapCenter.x) >> 1;  // Not: / 2

// GOOD: Pre-allocated arrays (avoid allocation in hot path)
private static final int[] KING_TILE_DX = {-1, 0, 1, -1, 1, -1, 0, 1};

// GOOD: Cache computed values across turn
private static boolean cachedIsAssassin = false;
private static int lastIsAssassinID = -1;

// GOOD: Deferred sensing (only sense when needed)
RobotInfo[] allies = null;  // Sense later only if overkill check needed

// BAD: Repeated sensing
for (int i = 0; i < enemies.length; i++) {
    RobotInfo[] nearby = rc.senseNearbyRobots(10, team);  // WASTEFUL
}
```

**Profile regularly**: Use `Clock.getBytecodeNum()` before/after expensive operations to track usage.

### State Machine Combat Pattern (ratbot5)

ratbot5 uses **kiting state machine** for combat:
```
APPROACH → ATTACK → RETREAT → (repeat)
```

**Key states**:
- `KITE_STATE_APPROACH (0)` - Move toward enemy until in attack range
- `KITE_STATE_ATTACK (1)` - Execute attack if action ready
- `KITE_STATE_RETREAT (2)` - Move away 1-3 tiles based on HP (wounded rats retreat farther)

**Critical features**:
- **Overkill prevention**: Don't attack enemies <20 HP if allies nearby (avoid wasted damage)
- **Focus fire**: King broadcasts priority target via shared array
- **Retreat coordination**: Broadcast when army HP ratio <40% of enemy
- **Dynamic kite distance**: Healthy (1 tile) → Wounded (2 tiles) → Critical (3 tiles)

### Shared Array Communication

**64 slots × 10 bits** (values 0-1023) for team coordination. DO NOT use squeaking (attracts cats).

**Common slot layout** (example from ratbot5):
```java
// Slot 0-1: Our king position (x, y)
// Slot 2-3: Enemy king position (x, y)
// Slot 4: Enemy king HP
// Slot 5: Economy mode (0=normal, 1=low, 2=critical)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/quanticsoul4772) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
