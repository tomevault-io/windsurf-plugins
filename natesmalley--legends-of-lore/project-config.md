---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🎮 Project: Legends of Lore - Gothic Retro Side-Scrolling Fantasy Game

A 2D side-scrolling action-adventure game inspired by Castlevania and Altered Beast, featuring retro pixel art graphics with a dark gothic aesthetic. The game follows Gabriel Thorne, a monster hunter with beast transformation abilities, on his quest to defeat Count Vladok.

## 🚨 IMPORTANT: Use CoralCollective Agents for Development

This project uses the CoralCollective framework for AI-driven development. ALWAYS use the specialized agents through the Task tool for different aspects of game development.

### Required Development Workflow:
1. **Start with Project Architect**: Design game architecture and systems
2. **Technical Writer Phase 1**: Document game requirements and specifications  
3. **Specialized Development**: Use appropriate agents for each component
4. **QA & Testing**: Comprehensive testing of game mechanics
5. **Technical Writer Phase 2**: Final documentation and guides

## Common Commands

### Game Development Setup
```bash
# Create virtual environment (ALWAYS DO THIS FIRST)
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies (when package.json/requirements.txt exist)
pip install -r requirements.txt  # Python dependencies
npm install                       # Node dependencies for web build

# Game engine setup (to be determined - Phaser.js recommended for web/mobile)
npm install phaser                # If using Phaser.js
```

### Running the Game
```bash
# Development server (once implemented)
npm run dev                       # Start development server
npm run build                     # Build for production
npm run test                      # Run tests

# Mobile builds (future)
npm run build:ios                 # Build for iOS
npm run build:android             # Build for Android
```

### Using CoralCollective Agents
```bash
# Access agent system
python .coral/agent_runner.py list              # List available agents
python .coral/agent_runner.py run               # Run specific agent
python .coral/agent_runner.py workflow           # Run workflow

# Use subagent invocation in Claude
@backend_developer "Create game server API"
@frontend_developer "Build game UI"
@ai_ml_specialist "Implement enemy AI"
```

## Architecture

### Game Architecture (To Be Implemented)

#### Core Systems
1. **Game Engine**: Phaser.js (recommended) or PixiJS for web/mobile compatibility
2. **Backend**: Node.js/Express API for:
   - User authentication & profiles
   - Save game management
   - Leaderboards
   - In-app purchases
   - Ad serving integration
3. **Database**: PostgreSQL or MongoDB for user data
4. **State Management**: Redux or Zustand for game state
5. **Mobile**: Capacitor or React Native for mobile builds

#### Game Components Structure
```
src/
├── game/                   # Core game code
│   ├── scenes/            # Game scenes/levels
│   │   ├── MainMenu.ts
│   │   ├── Level1_HauntedVillage.ts
│   │   ├── Level2_ForbiddenForest.ts
│   │   ├── Level3_Catacombs.ts
│   │   ├── Level4_ClockTower.ts
│   │   └── Level5_CastleKeep.ts
│   ├── entities/          # Game entities
│   │   ├── player/        # Player character
│   │   │   ├── Gabriel.ts
│   │   │   └── transformations/
│   │   ├── enemies/       # Enemy types
│   │   ├── bosses/        # Boss enemies
│   │   └── npcs/          # Non-player characters
│   ├── systems/           # Game systems
│   │   ├── combat/        # Combat mechanics
│   │   ├── transformation/# Beast transformation system
│   │   ├── progression/   # RPG elements
│   │   └── inventory/     # Items and weapons
│   ├── ui/                # Game UI components
│   │   ├── hud/          # Heads-up display
│   │   ├── menus/        # Game menus
│   │   └── dialogs/      # Dialog system
│   └── assets/            # Game assets
│       ├── sprites/       # Pixel art sprites
│       ├── audio/         # Music and SFX
│       └── levels/        # Level data
├── server/                 # Backend API
│   ├── routes/           # API endpoints
│   ├── models/           # Data models
│   └── services/         # Business logic
└── mobile/                 # Mobile-specific code
```

### Key Game Features to Implement

#### Phase 1: Core Gameplay
- Side-scrolling movement and combat
- Basic enemy AI
- Level progression system
- Save/load functionality

#### Phase 2: RPG Elements
- Experience and leveling system
- Skill tree implementation
- Weapon upgrades
- Item inventory

#### Phase 3: Transformation System
- Beast transformation mechanics
- Werewolf, dragon, specter forms
- Rage meter implementation
- Form-specific abilities

#### Phase 4: Monetization
- Free-to-play implementation
- $4.99/month subscription system
- Ad integration (AdMob/Unity Ads)
- In-app purchases
- Cosmetic skins and items

## Development Guidelines

### Game-Specific Patterns

#### Pixel Art Style
- Use 16-bit era inspired graphics
- Limited color palette for gothic aesthetic
- Consistent sprite sizes (32x32 or 64x64 base)
- Retro visual effects

#### Level Design Principles
- Gothic horror themes
- Platform challenges

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/natesmalley) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
