---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

AI生态小镇 (AI Eco Town) is a multi-agent simulation system based on Stanford's "Generative Agents" research paper. It simulates autonomous AI agents with memory, reflection, and planning capabilities living in a virtual 2D world.

**Architecture Note:** The project was refactored to a web-first architecture. The simulation now runs entirely in the browser, and the Node.js server is a lightweight `http.createServer` app for LLM API proxying and static file serving. `ARCHITECTURE.md` is outdated (still describes the old CLI architecture) — rely on this file and the source code instead.

## Common Commands

```bash
# Initialize runtime folders and .env
npm run setup

# Check whether this computer can run the project
npm run doctor

# Start web server (default) - runs on port 3061
npm start

# Development with hot reload (uses tsx watch)
npm run dev

# Build TypeScript
npm run build

# Run tests (vitest is installed but no test files exist yet)
npm test

# Lint
npm run lint

# Stop server (uses the local /api/stop endpoint; defaults to port 3061)
npm run stop
```

Note: vitest uses default configuration (no `vitest.config.*` file exists). The `data/` directory contains the SQLite database file (`ai-town.db`) and save files.
Prefer the npm scripts above as the source of truth; the Windows `.bat` helpers are legacy wrappers and can drift from the current scripts.
Use `npm ci` on fresh machines because `package-lock.json` is committed and is the reproducible dependency source of truth.

## Testing

There are no formal unit/integration tests. vitest is installed but no test files exist yet. When modifying frontend code, verify visually by running the server and opening `http://localhost:3061`.

## Architecture

### Current Architecture (Web-First)

The system has been refactored from a CLI-based simulator to a browser-based simulation:

1. **Server** (`src/server/`) - raw `http.createServer` HTTP server with manual route matching:
   - `index.ts` - Entry point, mounts middleware and routes
   - `routes/llm.ts` - Proxies LLM requests to configured provider
   - `routes/agents.ts` - Agent CRUD and actions
   - `routes/memories.ts` - Memory retrieval and storage
   - `routes/reflections.ts` - Reflection generation
   - `routes/map.ts` - Map data and building management
   - `routes/state.ts` - Simulation state persistence (including snapshot save/load)
   - `routes/sprites.ts` - Character sprite/portrait upload and listing
   - `middleware/json.ts` - JSON body parser
   - `middleware/multipart.ts` - Multipart form handling

2. **Database** (`src/server/db/`) - SQLite via better-sqlite3:
   - `connection.ts` - Database connection singleton
   - `schema.ts` - Table definitions (agents, memories, embeddings, reflections, areas, area_cells, simulation_state, dialogues, reflection_sources)

3. **Browser-Based Simulation** (`public/js/`) - Frontend organized by domain:
   - `core/agent.js` - `Agent` class: perception, decision-making, action execution
   - `core/simulator.js` - `WorldSimulator` class: 2D grid simulation with tick-based timing, pollution, resources, building levels
   - `core/memory.js` - `MemorySystem` class: three-layer memory (observations, reflections, plans)
   - `core/pathfinder.js` - A\* pathfinding with terrain collision
   - `core/personality.js` - Prompt construction and personality weight calculation
   - `core/prompts.js` - All LLM prompt templates in one place (imported by personality.js and memory.js)
   - `core/game-config.js` - Centralized game tuning constants (thresholds, rates, multipliers, pollution, building levels)
   - `app/app.js` - Main simulation logic, UI, event handling (canvas renderer)
   - `app/llm-client.js` - Communicates with backend `/api/llm/chat` endpoint
   - `assets/asset-config.js` - Sprite paths and display sizes
   - `assets/image-loader.js` - Asset loading manager
   - `editor/building-editor.js` - Map editing tools

   **Module dependency graph** (ES modules, all imported by `app/app.js`):

   ```
   app/app.js → core/simulator.js → core/agent.js → core/memory.js → core/prompts.js
                                            → core/pathfinder.js
                                            → core/personality.js → core/prompts.js
                                            → core/game-config.js
                 → app/llm-client.js
                 → assets/image-loader.js
                 → assets/asset-config.js
   ```

### Movement System

Agent movement is now independent of the simulation tick:

- **Tick interval** (`TICK_INTERVAL_MS`): Controls decision-making frequency (default: 5000ms)
- **Move interval**: 200ms per grid cell (independent timer in Agent)
- Agents make new decisions every 50 ticks OR when reaching their destination
- Movement uses A\* pathfinding (`pathfinder.js`) with terrain collision detection
- Path recalculates dynamically when blocked

### Memory Retrieval Algorithm

The system uses a three-dimensional weighted scoring for memory retrieval:

```
score = relevance × 0.6 + recency × 0.2 + importance × 0.2
```

Where:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yurifox123/ai-town-portable](https://github.com/yurifox123/ai-town-portable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
