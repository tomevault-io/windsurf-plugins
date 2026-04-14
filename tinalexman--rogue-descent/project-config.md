---
trigger: always_on
description: You are working on "Rogue Descent", a 2D side-scrolling multiplayer battle royale dungeon crawler built with Flutter and Flame engine. The game features 8-12 players exploring procedurally generated dungeons, setting traps, fighting NPCs, and converging for final PvP combat.
---

# Rogue Descent - Cursor Rules

## Project Overview
You are working on "Rogue Descent", a 2D side-scrolling multiplayer battle royale dungeon crawler built with Flutter and Flame engine. The game features 8-12 players exploring procedurally generated dungeons, setting traps, fighting NPCs, and converging for final PvP combat.

## Development Guidelines

### Code Style & Architecture
- Use Flutter/Dart best practices with proper async/await patterns
- Follow Flame engine component architecture (prefer composition over inheritance)
- Use Provider/Riverpod for state management
- Implement clean architecture with separate layers: presentation, domain, data
- Use meaningful variable names (avoid single letters except for loops)
- Add comprehensive comments for complex game logic
- Prefer immutable data structures where possible

### File Structure
```
lib/
├── core/           # Core game utilities, constants
├── data/           # Data models, repositories
├── domain/         # Business logic, entities
├── presentation/   # UI, game screens, widgets
├── game/           # Flame game components
│   ├── components/ # Game entities (player, enemies, traps)
│   ├── systems/    # Game systems (combat, networking)
│   └── worlds/     # Level generation, dungeon management
└── services/       # External services (networking, audio)
```

### Game-Specific Constants
Always use these exact values from the GDD:

#### Character Stats
- Warrior: 120 HP, 2.5 speed
- Archer: 80 HP, 3.0 speed  
- Rogue: 90 HP, 3.5 speed
- Mage: 70 HP, 2.8 speed

#### Timing Values
- Match duration: 15-20 minutes total
- Exploration phase: 10 minutes
- Convergence phase: 5-10 minutes
- Final battle: 2-5 minutes
- Dungeon collapse: starts at 10 minutes, 1 tile per 5 seconds

#### Damage Values
- Spike Pit: 30 damage + 2s slow
- Poison Dart: 15 damage + 10 DOT over 5s
- Explosive Mine: 40 damage, 2 tile radius
- Weak NPC kill: +10 XP
- Player elimination: +100 XP

### Networking Requirements
- Use WebSocket for real-time multiplayer
- Implement client-server architecture with authoritative server
- Target 20 updates per second for state synchronization
- Validate all actions server-side for anti-cheat
- Implement lag compensation and client prediction

### Performance Standards
- Maintain 60 FPS on mid-range mobile devices
- Keep memory usage under 150MB
- Network latency under 100ms
- Load times under 10 seconds

## Code Patterns

### Component Structure
```dart
class PlayerComponent extends SpriteAnimationComponent with HasKeyboardHandlerComponents, HasCollisionDetection {
  // Use this pattern for all game entities
}
```

### State Management
```dart
// Use Riverpod providers for game state
final gameStateProvider = StateNotifierProvider<GameStateNotifier, GameState>((ref) {
  return GameStateNotifier();
});
```

### Networking Messages
```dart
// Use consistent JSON structure for all network messages
class NetworkMessage {
  final String type;
  final String playerId;
  final Map<String, dynamic> data;
  final int timestamp;
}
```

## Asset Guidelines

### Sprite Specifications
- All sprites: 32x32 pixels
- Pixel art style, consistent color palette
- Character animations: 8 frames per action
- NPC animations: 4 frames per action
- Use sprite sheets for efficient memory usage

### Audio Requirements
- 3 ambient music tracks (menu, exploration, combat)
- 40+ sound effects total
- Use flame_audio package
- Compress audio files for mobile optimization

## Game Logic Rules

### Trap System Implementation
- Traps are invisible to all players except setter (show as semi-transparent)
- Maximum 3 traps per player active at once
- 60-second despawn timer on all traps
- Disarming takes 2 seconds of stationary interaction
- Traps don't affect the player who set them

### Level Generation
- Use grid-based system: 40x20 cells, each cell = 5x5 tiles
- Starting chambers: 15x10 tiles each, 40-60 tiles from center
- Generate paths using modified A* with room creation
- Add vertical shafts at path intersections for multi-level navigation
- Ensure all starting chambers have equal travel time to center

### Combat System
- All attacks have 0.5 second animation + recovery time
- Projectiles travel at 5 tiles/second
- Physical damage reduced by armor, magic damage bypasses armor
- Status effects: Slow (-50% speed), Poison (DOT), Stun (immobilized)

## Testing Requirements

### Unit Tests
- Test all damage calculations
- Test trap placement and triggering logic
- Test level generation algorithms
- Test networking message serialization

### Integration Tests  
- Test full gameplay loops
- Test multiplayer synchronization
- Test dungeon collapse mechanics
- Test cross-platform compatibility

### Performance Tests
- Profile on lowest target device spec
- Test with maximum player count (12 players)
- Test memory usage during long sessions
- Test network performance under poor conditions

## Security & Anti-Cheat

### Server Validation
- Validate all player movements on server
- Check damage calculations server-side
- Verify trap placement is legal

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tinalexman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
