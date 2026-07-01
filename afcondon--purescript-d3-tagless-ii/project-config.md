---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a PureScript library implementing a Finally Tagless embedded DSL for building interactive data visualizations. The library wraps D3.js functionality in a purely functional, composable API with strong type safety. The project targets PureScript 0.15.

## Repository Structure

This is a **Spago monorepo** with four packages:

```
├── psd3-selection/       # Core D3 selection/attribute library (publishable)
│   ├── spago.yaml        # Package config only
│   └── src/PSD3/         # Selection, attributes, layouts, interpreters
├── psd3-simulation/      # Force simulation library (publishable)
│   ├── spago.yaml        # Depends on psd3-selection
│   └── src/PSD3/         # ForceEngine, Config
├── psd3-tidal/           # TidalCycles mini-notation parser (publishable)
│   ├── spago.yaml        # Standalone package
│   └── src/Tidal/        # AST, Parser, Pretty-printer
├── demo-website/         # Documentation and examples website
│   ├── spago.yaml        # Depends on all libraries
│   ├── src/              # Halogen components, visualizations
│   └── public/           # Static assets, bundle.js
└── spago.yaml            # Workspace-only config (no package section)
```

## Build Commands

- **Install dependencies**: `npm install`
- **Build all packages**: `npm run build`
- **Build individual packages**:
  - `npm run build:selection`
  - `npm run build:simulation`
  - `npm run build:website`
- **Bundle website**: `npm run bundle` (outputs to `demo-website/public/bundle.js`)
- **Serve website**: `npm run serve` (http://localhost:1234)
- **Dev workflow**: `npm run dev` (build + bundle)

**Important**: This repo uses Spago 0.93 (via npm). Configuration is in `spago.yaml` files (one per package + workspace root).

## Architecture

### Finally Tagless Pattern

The library uses a Finally Tagless encoding that allows multiple interpreters for the same DSL. The core type classes define capabilities without tying them to specific implementations:

- **`SelectionM` capability**: Defines operations for DOM manipulation via selections (appending, selecting, joining data to DOM elements)
- **`SimulationM` capability**: Extends SelectionM with physics simulation capabilities for force-directed graphs

### Interpreters

Three interpreters demonstrate the pattern:

1. **D3 Interpreter** (psd3-selection/src/PSD3v2/Interpreter/D3v2.purs): Primary interpreter using D3.js via FFI
2. **MetaTree Interpreter** (psd3-selection/src/PSD3v2/Interpreter/MetaAST.purs): Generates visualizations of the DSL syntax tree itself
3. **English Interpreter** (psd3-selection/src/PSD3v2/Interpreter/English.purs): Generates human-readable descriptions

### psd3-selection Package

Core library for D3 selections, attributes, and layouts:

**Selection API** (PSD3v2/Selection/):
- `Types.purs`, `Operations.purs`, `Query.purs`, `Join.purs`
- Defines typed selection operations and the General Update Pattern

**Attributes** (PSD3v2/Attribute/, PSD3/Internal/Attributes/):
- Type-safe attribute setters with phantom types

**Layouts** (PSD3/Layout/):
- `Hierarchy/`: Tree, cluster, pack, partition, treemap layouts
- `Sankey/`: Sankey diagram layout

**Data Structures** (Data/, PSD3/Data/):
- `Tree.purs`, `Graph.purs`, `Node.purs`: Type-safe data structures
- Graph algorithms and dependency graph utilities

### psd3-simulation Package

Force-directed graph simulation:

**ForceEngine** (PSD3/ForceEngine/):
- `Core.purs/.js`: D3 force simulation wrapper
- `Simulation.purs`: High-level simulation API
- `Types.purs`: Force and simulation types

**Configuration** (PSD3/Config/):
- `Force.purs`: Immutable force configuration
- `Scene.purs`: Scene definitions
- `Apply.purs`: Apply configs to simulations

### psd3-tidal Package

TidalCycles mini-notation parser for visual pattern editing:

**Core** (Tidal/Core/):
- `Types.purs`: Time (Rational), SourceSpan, Seed, ControlName

**AST** (Tidal/AST/):
- `Types.purs`: TPat data type with 14 constructors (Atom, Silence, Seq, Stack, Fast, Slow, Polyrhythm, Euclid, etc.)
- `Pretty.purs`: Round-trip serialization back to mini-notation strings

**Parser** (Tidal/Parse/):
- `Class.purs`: AtomParseable class with instances for String, Number, Int, Rational
- `Combinators.purs`: Parser implementation using purescript-parsing
- `Parser.purs`: Entry points (parseTPat, parseMini)
- `State.purs`: Parser state for deterministic seed generation

**Usage**:
```purescript
import Tidal.Parse.Parser (parseTPat)
import Tidal.AST.Pretty (pretty)

-- Parse mini-notation
let ast = parseTPat "bd sn [hh hh]*2" :: Either _ (TPat String)

-- Round-trip back to string
let src = pretty <$> ast  -- Right "bd sn [hh hh]*2"
```

See `notes/TIDAL_VISUAL_EDITOR_GUIDE.md` for integration with PSD3 visual editing.

### demo-website Package

Halogen web application demonstrating the libraries:

**Structure** (demo-website/src/):
- `Main.purs`, `RoutingDSL.purs`, `Types.purs`: Top-level application
- **Component/**: Halogen components (CodeExplorerV3, ForceControlPanel, etc.)
- **HTML/**: Reusable HTML helpers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [afcondon/purescript-d3-tagless-II](https://github.com/afcondon/purescript-d3-tagless-II) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
