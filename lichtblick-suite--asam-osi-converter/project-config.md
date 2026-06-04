---
trigger: always_on
description: A Lichtblick extension that converts ASAM Open Simulation Interface (OSI) messages into 3D visualizations.
---

# ASAM OSI Converter Extension for Lichtblick

A Lichtblick extension that converts ASAM Open Simulation Interface (OSI) messages into 3D visualizations.

## Related Agent Docs
| Document | Purpose | Location |
| --- | --- | --- |
| Repository Guidelines | Contributor guide with commands, testing, and PR expectations | `AGENTS.md` |

## Build, Test, and Lint Commands

```bash
# Development
yarn install              # Install dependencies
yarn build                # Build extension
yarn local-install        # Build and install into local Lichtblick app

# Package for distribution
yarn package              # Creates .foxe file

# Testing
yarn test                 # Run all tests
yarn test <filename>      # Run specific test file (e.g., yarn test trafficlights.spec.ts)

# Linting
yarn lint                 # Auto-fix lint errors
yarn lint:ci              # Lint without auto-fix (CI mode)
```

## Architecture Overview

This is a **Lichtblick extension** that registers message converters to transform OSI protocol buffer messages into Foxglove visualization schemas.

### Core Components

**Extension Entry Point** (`src/index.ts`)
- Registers message converters using `extensionContext.registerMessageConverter()`
- Converts three OSI message types:
  - `osi3.GroundTruth` → `foxglove.SceneUpdate` + `foxglove.FrameTransforms`
  - `osi3.SensorView` → `foxglove.SceneUpdate` + `foxglove.FrameTransforms`
  - `osi3.SensorData` → `foxglove.SceneUpdate` + `foxglove.FrameTransforms`

**Converters** (`src/converters/`)
- `groundTruth/`: Main converter for ground truth data, produces SceneUpdate with entities
- `sensorView/`: Converter for sensor view data
- `sensorData/`: Converter for sensor-specific data
- Each converter has: `sceneUpdateConverter.ts`, `frameTransformConverter.ts`, `panelSettings.ts`, `context.ts`

**Features** (`src/features/`)
- Each feature module builds specific OSI entity types into Foxglove scene entities:
  - `movingobjects/` - Vehicles, pedestrians, animals (with light states)
  - `stationaryobjects/` - Static scene objects
  - `lanes/` - Lane geometries and boundaries
  - `logicallanes/` - Logical lane definitions
  - `trafficlights/` - Traffic light states and positions
  - `trafficsigns/` - Traffic signs with dynamic texture loading
  - `referenceline/` - Reference line geometries
  - `roadmarkings/` - Road marking visualizations
- Each feature exports builder functions like `buildMovingObjectEntity()`, `buildTrafficLightEntity()`, etc.

**Utils** (`src/utils/`)
- `primitives/`: Converts OSI objects to Foxglove primitives (cubes, models, lines)
- `scene.ts`: Scene entity management, ID generation, entity diffing
- `math.ts`: Mathematical transformations (Euler to quaternion, etc.)
- `helper.ts`: Color codes, timestamp conversion, path utilities
- `hashing.ts`: Entity hashing for change detection

**Config** (`src/config/`)
- `constants.ts`: All visualization constants (colors, sizes, materials) organized by feature
- `entityPrefixes.ts`: String prefixes for entity IDs (e.g., `PREFIX_LANE`, `PREFIX_TRAFFIC_LIGHT`)
- `frameTransformNames.ts`: Coordinate frame naming constants

### Message Converter Pattern

Converters follow this structure:
1. Create a context object to maintain state across conversions
2. Extract entities from the OSI message
3. Build Foxglove scene entities using feature builders
4. Track entity lifecycle (additions/deletions) for efficient updates
5. Return SceneUpdate or FrameTransforms schema

The converter function is returned as a closure that captures the `GroundTruthContext` — this context persists across frames and holds caches, previous entity ID sets, and the last known panel config. It is created once per `register*Converter()` call.

**SensorView** delegates to the GroundTruth converter using `msg.global_ground_truth`. **SensorData** is currently limited — it only renders detected lane boundaries and displays a "not supported yet" text label.

Panel settings flow into the converter via `event.topicConfig` (typed as `GroundTruthPanelSettings`). Converters fall back to `DEFAULT_CONFIG` when `topicConfig` is undefined.

Error handling: all converters wrap their main logic in `try/catch`, log with `console.error`, and return an empty `SceneUpdate` on failure rather than propagating exceptions.

### Feature Builder Pattern

Each feature module exports:
- `build*Entity()` - Creates a `PartialSceneEntity` with primitives
- `build*Metadata()` - Creates metadata for panel displays
- Uses utilities from `@utils/primitives` to create geometric primitives
- Returns entities with unique IDs generated via `generateSceneEntityId(prefix, id)`

## Path Aliases

TypeScript and Jest are configured with path aliases:

```typescript
@utils/*       → src/utils/*
@assets/*      → assets/*
@converters    → src/converters/index.ts
@converters/*  → src/converters/*
@features/*    → src/features/*
@/*            → src/*
```

Prefer these aliases for imports. They're configured in `tsconfig.json`, `jest.config.js`, and `config.ts`.

## Key Conventions

### Entity ID Generation
- Use `generateSceneEntityId(prefix, id)` from `@utils/scene` — produces `"${prefix}_${id}"`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lichtblick-suite/asam-osi-converter](https://github.com/lichtblick-suite/asam-osi-converter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
