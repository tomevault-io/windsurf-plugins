---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **complete 3D birthday game** created as a birthday present for Pea, who is turning 29 and loves knitting and law. The game features a character version of herself in third-person mode, navigating three rooms to complete challenges and unlock birthday videos.

### Game Concept
- **Player Character**: Pea - detailed 3D character with medium-length dark blonde hair
- **Companion**: Roma the cocker spaniel - intelligent AI companion that follows the player
- **Three Rooms**: Main room (starting area), Law room (courtroom quiz), Knitting room (crafting minigame)
- **Two Challenges**: Complete both to unlock the birthday chest
  1. **Law Quiz**: 9 basic UK law questions (Contract, Tort, Company law) in a courtroom setting
  2. **Knitting Minigame**: Create 3 pieces of knitwear (sweater, beanie, gloves) with materials and methods
- **Victory Condition**: Open chest to play personalized birthday videos 

## Development Setup

### Tech Stack
- **Three.js** - 3D graphics library for web-based game development
- **Vite** - Fast build tool and dev server
- **HTML5/CSS3** - Core web technologies
- **Vanilla JavaScript** - Keeping it simple without frameworks

### Commands
```bash
# Install dependencies
npm install

# Run development server (opens at localhost:3000)
npm run dev

# Build for production
npm run build
```

## Architecture

### Project Structure
```
Birthday/
├── index.html          # Main entry point
├── package.json        # Project dependencies
├── vite.config.js      # Vite configuration
├── src/
│   ├── main.js         # Game initialization
│   ├── game/
│   │   ├── Game.js     # Main game controller
│   │   ├── Player.js   # Player character controller
│   │   └── Camera.js   # Camera controls
│   ├── scenes/
│   │   ├── MainRoom.js # Starting room scene
│   │   ├── LawRoom.js  # Law quiz room
│   │   └── KnitRoom.js # Knitting minigame room
│   ├── minigames/
│   │   ├── LawQuiz.js  # Law quiz logic
│   │   └── Knitting.js # Knitting game logic
│   └── utils/
│       └── AssetLoader.js # Load 3D models and textures
├── assets/
│   ├── models/         # 3D models (character, furniture)
│   ├── textures/       # Textures and images
│   ├── videos/         # Birthday videos
│   └── sounds/         # Sound effects
└── style.css           # Basic styling

## Development History

**Development completed across 13 sessions** - detailed logs available in `DEVELOPMENT_HISTORY.md` if needed.

**Sessions 1-4**: Core game development (characters, rooms, minigames, video player)
**Sessions 5-7**: UI polish, Roma companion, bug fixes, progress tracking  
**Sessions 8-9**: Video player system, background music implementation
**Sessions 10-12**: Enhanced courtroom NPCs, dynamic chat bubbles, atmospheric animations
**Session 13**: Quick enhancements - .bat startup file, particle confetti system, enhanced lighting

**Current Status**: Production-ready with known issues being debugged.

### Session 13 - Quick Enhancements & Bug Fixes (In Progress)
**Completed:**
- ✅ Created `start-game.bat` file for easy one-click game startup
- ✅ Enhanced warm lighting across all rooms (brighter ambient, warmer colors, additional point lights)
- ✅ Integrated particle confetti system (150 confetti + 20 hearts + 50 sparkles on chest open)

**Known Issues (Debugging in Progress):**
- ⚠️ Confetti particles not rendering visually (system exists but particles not visible)
  - Added debug red cube to verify particle system
  - Console logging for particle creation and updates
  - Increased particle sizes and adjusted spawn positions
- ⚠️ Player movement locked after opening chest/closing video
  - Fixed property mismatch (`lockMovement` vs `isMinigameActive`)
  - Added key reset and pointer lock re-request
  - Console logging for movement state changes

**Debug Changes Made:**
- ParticleSystem.js: Added test red cube spawn, increased particle sizes, added debug logging
- Game.js: Fixed movement lock property, added pointer lock re-request, world position fixes
- All rooms: Added warm lighting methods with multiple point lights

**Next Session TODO:**
1. Verify if red test cube appears (indicates particle system works)
2. Check console for particle creation logs
3. Test if particles need different render order or material settings
4. Confirm movement unlock works with pointer lock re-engagement

## 🎉 ENHANCED COMPLETE GAME VERSION 🎂

**Status: PRODUCTION READY - ENHANCED ATMOSPHERIC GAME**

As of Session 12 completion, this represents the **enhanced atmospheric game** with dynamic courtroom chatter, living main room environment, and comprehensive animation systems. This is the current production-ready version with all core features, audio experience, enhanced NPC interactions, dramatic plant animations, warm lighting atmosphere, and realistic character behaviors.

### **🎮 Complete Game Features:**

**Core Gameplay:**
- ✅ Professional 3D third-person character (Pea) with detailed model and animations
- ✅ Three fully designed rooms (Main, Law, Knitting) with collision detection
- ✅ Professional mouse-look camera system with pointer lock

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DSYJ94/Birthday-Video-Game-Gift-2025-](https://github.com/DSYJ94/Birthday-Video-Game-Gift-2025-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
