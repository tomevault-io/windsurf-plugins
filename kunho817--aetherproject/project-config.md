---
trigger: always_on
description: Provides complete CSS implementation including:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Reference

- **Project**: Aether - A fully-featured incremental game with Glare Layer complete
- **Status**: Glare Layer fully implemented and functional
- **Stack**: Vue.js 3 + TypeScript + Pinia + break_infinity.js (switched from break_eternity.js)
- **Architecture**: Component-based with comprehensive state management
- **Key Rule**: Test changes thoroughly before committing - complex interconnected systems

## Project Overview

Aether is a complex incremental game with strategic depth. The Glare Layer (first of 10 layers) is fully implemented with all major systems operational including resource production, reset mechanics, cosmic filaments, constellation navigation, and advanced features like Star Memory, Automation, and Special Events.

## Project Status

**Current State**: Glare Layer fully implemented with comprehensive game systems.

**Available Files**:
- `Aether_Project_Concept.md` - High-level overview of all 10 game layers
- `Aether_Project_Glare_Layer_Concept.md` - Comprehensive design document for the Glare Layer (24,633 lines)
- `UI_Design_Ref.html` - Complete UI/CSS reference with animations and styling
- `break_eternity.js` - Library for handling large numbers (already downloaded)
- `CLI_Ref.md` - Claude CLI reference documentation

## Technology Stack (Planned)

Based on the parent directory's CLAUDE.md, this should be implemented as:
- **Framework**: Vue.js 3
- **State Management**: Pinia
- **Number Handling**: break_eternity.js (for large numbers beyond JavaScript's limits)
- **Styling**: CSS with animations and custom properties

## Project File Structure

```
Aether/
├── src/
│   ├── components/           # Vue components
│   │   ├── game/            # Game-specific components
│   │   │   ├── StarMap.vue
│   │   │   ├── ResourceDisplay.vue
│   │   │   ├── FilamentOrbit.vue
│   │   │   ├── NebulaGrid.vue
│   │   │   └── RailRoadNetwork.vue
│   │   └── ui/              # UI components
│   ├── stores/              # Pinia stores
│   │   ├── resources.ts     # Resource management
│   │   ├── filaments.ts     # Cosmic Filaments system
│   │   ├── starburst.ts     # Reset mechanics
│   │   ├── nebula.ts        # Grid system
│   │   └── gameLoop.ts      # Main game loop
│   ├── utils/               # Utilities
│   │   ├── decimal.ts       # break_eternity wrapper
│   │   ├── formatting.ts    # Number formatting
│   │   └── calculations.ts  # Game calculations
│   ├── types/               # TypeScript types
│   └── assets/              # Images, styles
├── public/                  # Static assets
├── tests/                   # Test files
└── docs/                    # Design documents
    ├── Aether_Project_Concept.md
    └── Aether_Project_Glare_Layer_Concept.md

```

## Key Design Documents

### Aether_Project_Glare_Layer_Concept.md
Contains complete specifications for:
- 15 detailed sections covering all game mechanics
- Exact formulas for calculations (e.g., Starburst multiplier: base x2, cumulative with x1.1 per Starlight)
- Reset mechanics (Starburst at 1e100 Stardust, tier unlock conditions)
- All 12 constellation effects with penalties and synergies
- UI layout specifications (Star Map system)
- Progression timeline (50+ hours to Nova Layer)

### UI_Design_Ref.html
Provides complete CSS implementation including:
- Star Map layout with central star and orbiting filaments
- Nebula grid system styling
- Rail Road constellation network visualization
- Animation keyframes for pulsation effects
- Color schemes and visual hierarchy
- Responsive design considerations

## Game Architecture

### Core Systems (Glare Layer)

1. **Resource System**
   - Stardust (basic production)
   - Starlight (prestige currency)
   - Star Rail, Nebular Essence, Stellar Energy, Cosmic Fragment

2. **Production Systems**
   - Cosmic Filaments (10 tiers with hierarchical production)
   - Milestone system (bonuses at purchase thresholds)
   - Evolution system (3 stages per filament)

3. **Reset Mechanisms**
   - Starburst (soft reset)
   - Starlight (extensive soft reset)
   - Supernova (layer transition preparation)

4. **Advanced Systems**
   - Nebula grid system with pattern bonuses
   - Star Pulsation with 5 cycle states
   - Rail Road constellation navigation
   - Star Memory preservation system
   - Upgrade tree with 4 branches

## Development Commands

```bash
# Start development server
npm run dev

# Build for production  
npm run build
npm run preview

# Type checking
npm run type-check

# Note: Linting and formatting are not configured
# npm run lint outputs "No linting configured"
# npm run format outputs "No formatting configured"
```

## Implemented Architecture

### Core Store Structure (Pinia)
- **gameState.ts** - Central game state, resources, filaments, reset logic
- **gameLoop.ts** - Main game loop and time management
- **achievements.ts** - Achievement tracking and unlocks
- **automation.ts** - Automated purchasing and progression
- **condensation.ts** - Advanced resource condensation mechanics
- **evolution.ts** - Filament evolution system (3 stages per tier)
- **memory.ts** - Star Memory preservation across resets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kunho817) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
