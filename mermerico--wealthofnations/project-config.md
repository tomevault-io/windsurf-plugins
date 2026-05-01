---
trigger: always_on
description: **CRITICAL**: Always follow this workflow:
---

# Wealth of Nations - Project Structure Guide

## Development Workflow

**CRITICAL**: Always follow this workflow:

1. **Run Tests First**: Execute `npm run test` to verify all tests pass
2. **Build the Application**: Run `npm run build` to check for TypeScript/build errors
3. **Avoid Running Dev Server**: Do NOT run `npm run dev` - the user will test functionality. Focus on ensuring tests pass and build succeeds.
4. **Playwright Testing**: When running Playwright tests, ensure the HTML report is disabled (e.g., use `--reporter=line`) so that the command exits immediately upon completion instead of hanging to serve the report.

This ensures you catch errors early and avoid debugging blank pages or runtime issues that could have been caught at build time. Let the user handle running the dev server and testing the UI.

## Overview

This project is a web-based implementation of the board game **Wealth of Nations**, a game of industrial production and international trade. The current implementation supports local hotseat multiplayer (players take turns on the same device). The game is built using React, TypeScript, and Vite.

## Technology Stack

- **Frontend Framework**: React 19 with TypeScript
- **Build Tool**: Vite
- **Testing**: Vitest with React Testing Library
- **Styling**: CSS (component-level)
- **Development**: ESLint for code quality

## Project Architecture

### Core Structure

```
### Core Structure

```
.
├── src/
│   ├── types/          # TypeScript type definitions
│   ├── utils/          # Game logic utilities
│   ├── hooks/          # React hooks (game state management)
│   ├── components/     # React components (UI and game elements)
│   ├── pages/          # Page-level components
│   └── tests/          # Integration and unit tests
│       └── integration/ # Complex interaction tests
├── e2e/                # Playwright End-to-End tests
├── public/             # Static assets (flag SVGs)
├── resources/          # Game rules and reference materials
└── config files        # TypeScript, Vite, ESLint configurations
```

## Core Game Concepts

### Game Mechanics (from resources/rules.md)


The game revolves around three main phases that repeat each round:

1. **Trade Phase**: Players buy and sell commodities on markets
2. **Develop Phase**: Players place flags and build industry tiles on the hexagonal board
3. **Produce Phase**: Industries produce commodities based on their configuration

**Key Resources**:
- **Commodities**: Food (yellow), Energy (blue), Labor (red), Ore (gray), Capital (black)
- **Money**: Purple resource used for trading and scoring
- **Victory Points**: Final scoring mechanism

**Key Mechanics**:
- Hexagonal grid-based board
- Industry tiles with dots (production units) and partial dots that combine with adjacent tiles
- Market system with dynamic pricing based on supply/demand
- Bloc system: connected tiles of the same type produce together
- Automation tokens for enhanced production

## Detailed File Structure

### `/src/types/gameState.ts`

**Purpose**: Central type definitions for the entire game state.

**Key Types**:
- `CommodityType`: Union type for the 5 commodities
- `IndustryType`: Union type for the 6 industry types (Farm, Generator, Academy, Mine, Factory, Bank)
- `Player`: Player data including resources, money, loans, flags
- `HexCell`: Individual hex on the board with coordinate and occupant
- `IndustryTile`: Tile instance with type, owner, orientation, active state, automation
- `TileFeature`: Visual/functional features on tile edges/corners (half-dots, third-dots)
- `MarketState`: Stock level and price index for each commodity market
- `GameState`: Complete game state including players, board, markets, phase, turn order

**Design Pattern**: Immutable state structure designed for React useState/useReducer patterns.

### `/src/hooks/useGameEngine.ts`

**Purpose**: Main game state management hook. This is the "engine" that drives the game.

**Key Responsibilities**:
- Maintains the complete `GameState` using React `useState`
- Provides `handleAction()` function for all game actions
- Manages phase transitions (Trade → Develop → Produce)
- Manages turn order and player switching
- Implements game actions:
  - `pass`: Skip turn, advance to next player/phase
  - `placeFlag`: Place a player flag on an empty hex
  - `placeTile`: Build an industry tile on a flagged hex
  - `buy`/`sell`: Market transactions
  - `rotate`/`move`: Manipulate tiles during Develop phase
  - `automate`: Add automation token to a tile
  - `produce`: Execute production for a bloc
  - `delete`: Remove tiles/flags

**State Management Pattern**: 
- Uses functional setState updates for immutability
- Validates actions before applying state changes
- Returns success/failure results for UI feedback

**Key Functions**:
- `nextTurn()`: Cycles to next player
- `nextPhase()`: Advances game phase
- `handleAction()`: Central dispatcher for all game actions

### `/src/utils/` Directory

#### `hexUtils.ts`
**Purpose**: Hexagonal grid mathematics and coordinate system.

**Key Functions**:
- `generateGrid(radius)`: Creates the hexagonal board
- `coordsToString(q, r)`: Converts coordinates to string keys
- `stringToCoords(key)`: Parses coordinate strings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mermerico/WealthOfNations](https://github.com/mermerico/WealthOfNations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
