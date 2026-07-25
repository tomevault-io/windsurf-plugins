---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

- **Build**: `bun dev:build --silent` - Builds the project using a custom Bun Build based script
  - `--silent` or `-s` suppresses progress output (this is what you should default to for context preservation)
- **Watch**: `bun dev:watch` - Builds with watch mode
- **Type checking**: `bun dev:build:types` - Generates TypeScript declaration files only
- **Lint**: `bun lint` - Lints TypeScript files using OxLint
- **Format/Fix**: `bun format` - Lints and auto-fixes issues

**Note**: The build scripts auto-retry on "Excessive complexity" TypeScript errors caused by stale types. If you see this error followed by "cleaning and retrying..." and the build succeeds, ignore it.

## Todo Directory

The `todo/` directory contains planning and tracking documents for ongoing development. Currently empty.

## Code Architecture

Sandstone is a TypeScript library for generating Minecraft datapacks and resource packs programmatically. It provides a type-safe, fluent API with ~7,500 lines of source code across 200+ TypeScript files.

### Directory Structure

```
src/
├── arguments/           # Type-safe argument definitions
│   ├── generated/       # Auto-generated types from Minecraft data
│   │   ├── _builtin/    # Built-in Minecraft types (block states, custom data)
│   │   ├── _dispatcher/ # Dispatcher types for polymorphic features
│   │   ├── _registry/   # Registry types (ITEMS, BLOCKS, ENTITY_TYPES, etc.)
│   │   ├── assets/      # Resource pack asset types
│   │   ├── data/        # Datapack data types (advancement, loot, recipes)
│   │   ├── util/        # Utility types (GlobalPos, WeightedList)
│   │   ├── world/       # World block, entity, item component types
│   │   ├── registry.ts  # Main Registry type export
│   │   ├── dispatcher.ts
│   │   ├── pack.ts
│   │   └── util.ts
│   ├── resources/       # Resource argument definitions
│   │   ├── datapack/    # Advancement, loot table, predicate args
│   │   └── resourcepack/ # Atlas, blockstate, font, model args
│   └── *.ts             # Core argument types (basics, nbt, selector, coords)
├── core/                # Core engine and AST infrastructure
│   ├── sandstoneCore.ts # Central engine managing resources and MCFunctions
│   ├── nodes.ts         # AST node base classes (Node, ContainerNode, CommandNode)
│   ├── visitors.ts      # Visitor pattern for AST transformations
│   ├── Macro.ts         # Macro argument system
│   └── resources/       # Resource implementations
│       ├── resource.ts  # Base ResourceClass and ResourceNode
│       ├── datapack/    # MCFunction, Advancement, LootTable, etc.
│       └── resourcepack/ # Atlas, BlockState, Texture, Font, etc.
├── pack/                # Pack management and build system
│   ├── pack.ts          # SandstonePack main class
│   ├── packType.ts      # PackType definitions (datapack/resourcepack)
│   └── visitors/        # AST transformation visitors
│       ├── ifElseTransformationVisitor.ts
│       ├── loopTransformationVisitor.ts
│       ├── inlineFunctionCallVisitor.ts
│       ├── simplifyExecuteFunction.ts
│       └── ...
├── commands/            # Minecraft command implementations
│   ├── commands.ts      # SandstoneCommands registry
│   └── implementations/ # Individual command classes (50+ commands)
│       ├── block/       # clone, fill, place, setblock
│       ├── entity/      # attribute, damage, effect, summon
│       ├── player/      # gamemode, give, particle
│       ├── server/      # datapack, reload, difficulty
│       └── world/       # seed, weather, time, locate
├── variables/           # Variable and expression system
│   ├── Objective.ts     # Scoreboard objective wrapper
│   ├── Score.ts         # Scoreboard score variable
│   ├── Selector.ts      # Entity selector with fluent builder
│   ├── Data.ts          # NBT data path navigation
│   ├── Coordinates.ts   # Coordinate systems (absolute, relative, local)
│   └── nbt/             # NBT manipulation utilities
├── flow/                # Control flow abstractions
│   ├── Flow.ts          # Main Flow class (if/else, loops, switch)
│   ├── conditions/      # Condition types and builders
│   └── ...
└── index.ts             # Main entry point exporting public API
```

### Core Components

#### AST System (`src/core/nodes.ts`)
- **Node**: Abstract base for all AST nodes with `sandstoneCore` context
- **ContainerNode**: Nodes containing a body of other nodes (functions, loops)
- **CommandNode**: Represents Minecraft commands with arguments
- **MCFunctionNode**: Represents Minecraft functions as containers with context stack

#### SandstoneCore (`src/core/sandstoneCore.ts`)
Central engine managing:
- `resourceNodes`: Map of all created resources
- `mcfunctionStack`: Stack tracking active MCFunction context
- `enterMCFunction()` / `exitMCFunction()`: Context switching
- `currentMCFunction`: Getter for active function

#### Resource System (`src/core/resources/resource.ts`)
- **ResourceClass**: Abstract base for all resources with conflict resolution
- **Conflict strategies**: throw, replace, ignore, append, prepend, rename

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sandstone-mc/sandstone](https://github.com/sandstone-mc/sandstone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
