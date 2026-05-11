---
trigger: always_on
description: > **Note**: This document is optimized for LLM consumption and provides comprehensive technical context for AI assistants working with the Noodles.gl codebase. For human-readable documentation, see [/docs](docs/) and [/dev-docs](dev-docs/).
---

# AGENTS.md - LLM Context for Noodles.gl

> **Note**: This document is optimized for LLM consumption and provides comprehensive technical context for AI assistants working with the Noodles.gl codebase. For human-readable documentation, see [/docs](docs/) and [/dev-docs](dev-docs/).

This document provides essential context for Large Language Models (LLMs) working with the Noodles.gl codebase.

## Project Overview

**Noodles.gl** is a React-based node-based editor for creating geospatial visualizations and animations. It combines visual programming with reactive data flow to build interactive presentations, high-quality renders, and data-driven animations.

### Core Purpose
- Create animated timeline presentations and data visualizations
- Specialize in geospatial data, aviation routes, and interactive storytelling
- Enable rapid prototyping and development of complex visualizations
- Export to video, images, or interactive web presentations

### Key Users
- Visualization experts creating presentation-ready graphics
- Developers doing rapid visualization prototyping
- Data scientists exploring and analyzing data
- Research teams publishing geospatial analysis

## Quick Reference Links

- **[Architecture](dev-docs/architecture.md)** - System architecture, state management, and error handling
- **[Development Guide](dev-docs/developing.md)** - Setup, commands, workflows, and best practices
- **[Testing Guide](dev-docs/testing-guide.md)** - Testing strategy, critical components, and runbook guidelines
- **[PR Guidelines](dev-docs/pr-guidelines.md)** - Creating focused PRs with tests and documentation
- **[Analytics](dev-docs/analytics.md)** - Privacy-preserving analytics guidelines
- **[Tech Stack](dev-docs/tech-stack.md)** - Complete technology listing

## Architecture

### Fundamental Concepts

**Operators**: Processing nodes that transform data
- Pure functions: deterministic (same inputs = same outputs)
- Reactive: automatically re-execute when upstream data changes
- Typed: use Zod schemas for input/output validation
- Memoized: results cached to avoid unnecessary recomputation

**Fields**: Typed inputs/outputs with validation and UI hints
- Strongly typed RxJS observables
- Support both value and reference connections
- Can be keyframed in timeline for animations
- Custom React components for specialized UI controls

**Pull-Based Execution**: Demand-driven operator execution
- Operators only execute when outputs are requested and inputs have changed
- Dirty flag system tracks which operators need re-execution
- Topological sorting determines execution order
- Parallel execution for independent branches
- GraphExecutor manages the execution loop with RAF-based timing

### Technology Stack

**Core:** React 18, TypeScript, Vite, Yarn

**Animation:** Native timeline system (bezier interpolation, keyframeable parameters)

**Visualization:** Deck.gl (WebGL data visualization), MapLibre GL (mapping), luma.gl (rendering), D3.js (data)

**Geospatial:** @turf/turf (analysis), H3-js (indexing), DuckDB-WASM (SQL), Apache Arrow (columnar data)

**UI:** @xyflow/react (node editor), Radix UI, PrimeReact

**State:** Zustand (global state, timeline state), RxJS (reactive data flow)

**Dev Tools:** Biome (linting/formatting), TypeScript, Vitest, Playwright

See [tech-stack.md](dev-docs/tech-stack.md) for complete details.

## Project Structure

```
noodles-gl-public/
├── noodles-editor/           # Main application
│   ├── src/
│   │   ├── noodles/          # Core node system
│   │   │   ├── operators.ts  # Operator registry
│   │   │   ├── fields.ts     # Field system
│   │   │   ├── components/   # React components
│   │   │   │   ├── op-components.tsx      # Operator node renderers
│   │   │   │   ├── field-components.tsx   # Field input renderers
│   │   │   │   ├── menu.tsx               # Operator menu
│   │   │   │   └── categories.ts          # Operator categorization
│   │   │   ├── utils/        # Utilities
│   │   │   │   ├── path-utils.ts          # Path resolution
│   │   │   │   ├── memoize.ts             # Caching
│   │   │   │   ├── serialization.ts       # Save/load
│   │   │   │   └── ...
│   │   │   └── hooks/        # React hooks
│   │   ├── ai-chat/          # Claude AI integration
│   │   ├── utils/            # General utilities
│   │   ├── timeline-editor.tsx  # Timeline interface
│   │   ├── noodles.tsx       # Main viz component
│   │   └── index.tsx         # App entry point
│   ├── public/
│   │   └── noodles/          # Example projects
│   ├── scripts/              # Build scripts
│   ├── package.json
│   ├── vite.config.js
│   └── tsconfig.json
├── website/                  # Documentation website
├── docs/                     # Documentation source
│   ├── developers/           # Developer guides
│   └── users/                # User guides
├── dev-docs/                 # Internal dev docs
│   ├── architecture.md
│   ├── developing.md
│   ├── testing-guide.md
│   ├── pr-guidelines.md
│   ├── analytics.md
│   ├── tech-stack.md
│   └── specs/                # Design specs
├── README.md
└── CONTRIBUTING.md
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joby-aviation/noodles.gl](https://github.com/joby-aviation/noodles.gl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
