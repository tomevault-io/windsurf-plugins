---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
AI-powered Werewolf game framework - a monorepo implementing an AI-driven multiplayer werewolf game with distinct AI personalities, featuring immersive TTS audio, background music, and real-time visualization.

🎮 **Online Demo**: https://solution.minimaxi.com/werewolf/

## Tech Stack & Package Manager
- **Package Manager**: Bun (no build step needed for backend, direct execution)
- **Frontend**: Vite + React + MobX + TailwindCSS
- **Backend**: Node.js/Bun + Express
- **AI Integration**: MiniMax AI (MiniMax-M2 model), Langfuse telemetry
- **Audio**: Web Audio API, TTS (Text-to-Speech), Background Music
- **State Management**: MobX with global stores
- **Deployment**: Docker + Docker Compose
- 我用bun，不需要build

## Critical Development Rules
- **TypeScript**: NEVER use `any` type - always use proper typing
- **Always use ultrathink** for complex reasoning tasks
- **Player IDs**: Always use numbers for Player IDs
- **Shared Types**: Only put types in shared/ if needed by Player services (e.g., API types called by game master)
- **MobX Reactivity**: ALL components using MobX state MUST use `observer` HOC
- **Audio Context**: TTS and BGM use synchronized AudioCoordinator for ducking

## Common Development Commands

### Docker Deployment (Production)
```bash
# Build and run with Docker Compose
docker-compose up -d

# View logs
docker-compose logs -f

# Stop containers
docker-compose down

# Rebuild after code changes
docker-compose up -d --build
```

### Development
```bash
# Start all 6 AI players (ports 3001-3006)
./scripts/dev-players.sh
# OR
bun run dev:players

# Start game master frontend (port 3000)
bun run dev:game-master

# Start individual player with config
bun run dev:player:aggressive
bun run dev:player:conservative
bun run dev:player:witty
bun run dev:player:default
```

### Code Quality
```bash
# Type checking (entire monorepo)
bun run typecheck
bunx tsc --build

# Type checking specific packages
bun run typecheck:frontend
bun run typecheck:backend

# Linting
bun run lint

# Testing (when tests exist)
bun test
bun run test:packages
bun run test:coverage
```

## Architecture Overview

### Monorepo Structure
```
packages/
├── game-master-vite/   # Frontend UI (Vite + React + MobX)
│   └── src/
│       ├── components/      # React components with observer HOC
│       │   ├── ImmersiveMode/  # Immersive view components
│       │   │   ├── ImmersiveView.tsx     # Main immersive mode
│       │   │   ├── EventTimeline.tsx     # Left panel: key events
│       │   │   ├── ThinkingPanel.tsx     # Right panel: inner thoughts
│       │   │   ├── PlayerCircle.tsx      # Circular player layout
│       │   │   └── PhaseIndicator.tsx    # Game phase display
│       ├── stores/          # MobX global stores
│       └── lib/             # GameMaster class, audio system
│           ├── GameMaster.ts         # Core game logic
│           ├── PlayerAPIClient.ts    # HTTP client for AI players
│           └── audio/                # Audio system
│               ├── AudioCoordinator.ts      # TTS + BGM coordinator
│               ├── TTSQueue.ts             # TTS playback queue
│               └── BackgroundMusicPlayer.ts # Phase-based BGM
├── player/             # AI player server
│   └── src/
│       ├── services/   # AIService, PersonalityFactory
│       ├── configs/    # Player personality configs (JSON)
│       └── prompts/    # Modular prompt templates
shared/
├── types/              # Shared TypeScript types & schemas
│   └── src/
│       ├── api.ts      # API request/response types
│       └── schemas.ts  # Zod schemas for AI responses
├── lib/                # Shared utilities & Langfuse integration
└── prompts/            # AI prompt templates
```

### Core Game Flow
1. **Game Creation**: Frontend calls `gameMaster.createGame(6)` → adds 6 AI players → assigns roles
2. **Game Phases**: Night (role abilities) → Day (discussion) → Voting → repeat
3. **AI Players**: Each runs on separate port (3001-3006), receives game state via HTTP API
4. **Role System**: 4 roles only - VILLAGER, WEREWOLF, SEER, WITCH (no HUNTER/GUARD)
5. **Immersive Mode**: TTS narration, background music, real-time event timeline, inner thoughts display

## MobX React Development Pattern

### Required Pattern
```typescript
// ✅ ALWAYS use this pattern
import { observer } from 'mobx-react-lite';
import { gameMaster } from '@/stores/gameStore';

export const Component = observer(function Component() {
  const data = gameMaster.computedProperty; // Direct global state access
  return <div>{data}</div>;
});
```

### Core MobX Rules
1. **Global State First**: Access state directly from global stores, never pass through props
2. **Observer Wrapper**: ALL components using MobX state MUST use `observer` HOC
3. **Computed Properties**: Use `computed` for derived data to optimize performance
4. **Avoid Redundant APIs**: Get data directly from state, don't make unnecessary network requests
5. **Observable GameLog**: `gameMaster.gameLog` is observable for real-time updates to EventTimeline/ThinkingPanel

## Immersive Mode Architecture

### Audio System

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mm-demo-collection/minimax-werewolf](https://github.com/mm-demo-collection/minimax-werewolf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
