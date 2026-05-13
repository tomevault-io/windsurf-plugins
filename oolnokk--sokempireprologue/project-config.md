---
trigger: always_on
description: This is a 2D/3D game engine toolkit for "SoK Empire Prologue" - a fighting game demo with extensive editor tools. The project uses vanilla JavaScript with Three.js for 3D rendering.
---

# SoK Empire Prologue - AI Assistant Guidelines

## Project Overview
This is a 2D/3D game engine toolkit for "SoK Empire Prologue" - a fighting game demo with extensive editor tools. The project uses vanilla JavaScript with Three.js for 3D rendering.

**IMPORTANT: Before implementing ANY feature, search for existing code first!**

## Quick Stats
- ~204 JS/JSON files
- 25MB total size
- No build step required (vanilla JS + ES modules)
- Offline-first with CDN fallbacks

---

## Directory Structure & What Already Exists

### `/docs/` - Main Game & Editor Tools
The primary codebase. Contains the live game demo and all editor tools.

#### Key HTML Files (Editors & Tools)
- `index.html` - Main game demo
- `animation-editor.html` - Animation frame editor
- `cosmetic-editor.html` - Character cosmetics editor
- `map-editor.html` - 2D map layout editor
- `gameplay-map-editor.html` - Gameplay logic editor (spawns, patrols, etc.)
- `3Dmapbuilder.html` - 3D map builder
- `structure-editor.html` - Structure/building editor
- `map-object-editor.html` - Map object editor
- `ability-ui-test.html` - Ability UI testing tool
- `hud-arch-demo.html` - HUD architecture demo

**BEFORE creating a new editor, check if one already exists above!**

#### `/docs/renderer/` - Core 3D Rendering System
**✅ COMPLETE SYSTEMS - DO NOT RECREATE:**
- `Renderer.js` - Main WebGL renderer class
- `scene3d.js` - 3D scene management
- `visualsmapLoader.js` - 3D asset loading with caching (has cache-busting for dev mode)
- `gltfTransforms.js` - GLTF model transformation utilities
- `rendererAdapter.js` - Adapter for renderer integration
- `index.js` - Renderer module exports

#### `/docs/config/` - Game Configuration Data
**Existing Config Systems:**
- `maps/visualsmaps/` - 3D map visual configurations (GLTF models, positions, rotations)
  - `index.json` - Main visual map registry
  - `defaultdistrict3D_visualsmap.json` - Example 3D district
- `maps/gameplaymaps/` - Gameplay logic (spawn points, patrol routes, collision)
  - `defaultdistrict3d_gameplaymap.json` - Example gameplay map
- `cosmetics/` - Character appearance items (clothing, hair, accessories)
  - `cosmetics/appearance/` - Species-specific appearance options
- `abilities/` - Character abilities and skills
- `assets/` - Asset configurations (roads, sidewalks, towers, etc.)
  - `asset-index.json` - Central asset registry
- `fighter-offsets/` - Character sprite offset data
- `prefabs/` - Reusable game object templates
- `config.js` - Main configuration module

**BEFORE adding new config types, check if a similar system exists!**

#### `/docs/assets/` - Game Assets
- `3D/` - GLTF 3D models
- `fightersprites/` - Character sprite sheets
- `weapons/` - Weapon graphics
- `cosmetics/` - Cosmetic item graphics
- `audio/` - Sound effects and music
- `areas/` - Background/area graphics
- `hud/` - UI/HUD elements
- `props/` - Prop graphics
- `prefabs/` - Prefab graphics

### `/src/` - Source Code (Alternative/Shared Modules)
Some systems have versions in both `/src/` and `/docs/`. Check both!

#### `/src/renderer/` - Renderer Source
- `Renderer.js` - Core renderer (also in docs/renderer/)
- `index.js` - Module exports

#### `/src/map/` - Map Systems
**✅ EXISTING MAP SYSTEMS:**
- `MapRegistry.js` - Map registration and management
- `GeometryService.js` - Geometry/collision calculations
- `scene3d.js` - 3D scene (alternative to docs version)
- `rendererAdapter.js` - Renderer integration
- `groupLibrary.js` - Reusable object groups
- `builderConversion.js` - Map builder data conversion
- `mapBuilderConfig.js` - Map builder configuration

#### `/src/spawn/` - Spawning System
**✅ COMPLETE SPAWN SYSTEM:**
- `SpawnService.js` - Entity spawning service

#### `/src/lighting/` - Lighting Systems
**✅ EXISTING LIGHTING FEATURES:**
- `DayNightSystem.js` - Day/night cycle
- `CandleLight.js` - Dynamic candle lighting
- `TowerLightingIntegration.js` - Tower-specific lighting

#### `/src/config/` - Config Modules
- `maps/` - Map configurations
- `groups/` - Group configurations

### `/docs/vendor/` - Third-party Libraries
**✅ THREE.JS v0.160.0 ALREADY INSTALLED:**
- `three/three.min.js` - Three.js core (classic build)
- `three/three.module.js` - Three.js ES module
- `three/GLTFLoader.js` - GLTF loader (classic)
- `three/GLTFLoader.module.js` - GLTF loader (ES module)
- `three/BufferGeometryUtils.js` - Geometry utilities
- See `docs/vendor/three/README.md` for details

**DO NOT re-download or reinstall Three.js - it's already set up with offline support!**

### Root Directory Files
- Multiple `.md` files documenting past PRs and implementations
- `ancient code-monolith of truth*.html` - Legacy monolithic code (mostly deprecated)
- `package.json` - Project metadata and scripts
- `eslint.config.mjs` - ESLint configuration

### `/tools/` - Build & Utility Scripts
- `githack-url.mjs` - Generate raw.githack.com preview URLs
- `copy-map-layout.mjs` - Map layout builder tool

### `/tests/` - Test Suite
Unit tests using Node's built-in test runner.

---

## Common Patterns & Conventions

### Module System
- **ES Modules** everywhere (`import`/`export`)
- Use `.js` extensions in import paths

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Oolnokk/SoKEmpirePrologue](https://github.com/Oolnokk/SoKEmpirePrologue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
