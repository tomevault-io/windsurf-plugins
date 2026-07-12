---
trigger: always_on
description: Build a polished, high-performance pixel-art asset tool for turning AI-generated “looks like pixel art” images into real, grid-aligned, palette-limited, engine-ready pixel-art assets.
---

# AGENTS.md

## Project mission

Build a polished, high-performance pixel-art asset tool for turning AI-generated “looks like pixel art” images into real, grid-aligned, palette-limited, engine-ready pixel-art assets.

Working name: **PixelAid**. Rename freely if the repo already has a better name.

The product should feel like a serious game-engine or art-tool editor: fast viewport, inspector panels, timelines, asset browser, exporters, preview modes, and precise technical controls. It should not feel like a toy image filter.

## Product principles

1. **The output must be real pixel art.** No fake enlarged pixels, no anti-aliased preview lies, no inconsistent grids hidden from the user.
2. **The UI must expose the grid.** Always show native output size, zoom level, grid confidence, palette count, frame size, alpha mode, and export metadata.
3. **Performance is a feature.** Keep the UI responsive during heavy processing. Rendering should be deliberately optimized, especially canvases, previews, sprite playback, and future 2D/3D sandbox views.
4. **Automation first, manual override always.** Auto-detect grid, palette, frames, and pivots, but every major detection result needs a clean manual override.
5. **Engine-ready assets, not just pretty images.** Exports must include pivots, frame rects, animation tags, durations, padding/extrusion, palette metadata, and target-engine guidance.
6. **Minimal dependencies by default.** Prefer small, permissive, well-maintained libraries. Avoid GPL/AGPL/LGPL or commercially restrictive dependencies unless explicitly isolated and documented.
7. **AI integrations are optional modules.** The core pixel-fixing engine must work without API keys, network calls, or vendor lock-in.

## Guiding Principles

### 1. Plan Before Coding — Always

Before writing any code:

1. State the goal in plain language.
2. Break it into a numbered list of small, independently testable tasks.
3. Present the plan to the user and **wait for explicit confirmation** before proceeding.
4. If any step is ambiguous or underspecified, ask a clarifying question (see §Clarifications).

Never combine planning and implementation in the same response.

### 2. Small, Testable Steps

Each task should:

- Produce a single coherent change (one new file, one refactor, one new system, etc.)
- Be verifiable — either via a Vitest test, a visible browser change, or a console confirmation.
- Be described with a clear "how to verify" note before implementation.

If a task feels too large to verify in one step, split it further.

### 3. Semantic Commits

Every change must be committed with a semantic commit message:

```
<type>(<scope>): <short description>

[optional body]
```

**Types:**
| Type | When to use |
|------|-------------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `test` | Adding or updating tests |
| `chore` | Build config, tooling, dependencies |
| `docs` | Documentation only |
| `perf` | Performance improvement |

**Scopes:** `core`, `worker`, `exporters`, `ai`, `cli`, `mcp`, `shared`, `fixtures`, `tooling`, `deps`, `core`, `web`, `desktop`, `docs`

**Examples:**
```
feat(ecs): add World class with entity creation and destruction
test(ecs): add vitest coverage for component registration
refactor(renderer): decouple ThreeRenderer from World lifecycle
chore(deps): add three and @types/three to engine package
```

Commit after each completed task — not in bulk at the end.

### 4. Clarifications — Ask, Never Assume

If any of the following are unspecified, **stop and ask** before proceeding:

- Naming that could conflict with Three.js built-ins
- Whether existing code should be refactored or extended
- Test coverage expectations for a given feature

Ask one focused question at a time. Do not batch unrelated clarifications into a single block unless they are all blockers for the same task.

## Expected application shape

Use a Vite + React + TypeScript web app with a pure image-processing core.

Preferred monorepo layout:

```txt
apps/
  web/                    # Vite + React editor UI
  desktop/                # Future Tauri shell wrapping the web app
packages/
  core/                   # Pure TS image algorithms; no React, no DOM
  worker/                 # Web Worker wrappers around core algorithms
  exporters/              # Godot, Unity, Phaser, TexturePacker, generic JSON
  ai/                     # Future optional AI-provider adapters
  cli/                    # Future Node CLI and batch processing entrypoint
  mcp/                    # Future MCP server exposing fix/export operations
  shared/                 # Shared types, constants, schemas
  fixtures/               # Sample/generated test images and golden outputs
docs/
  architecture.md
  algorithms.md
  performance.md
  licensing.md
```

The core package should operate on plain data structures:

```ts
export type RGBAImage = {
  width: number;
  height: number;
  data: Uint8ClampedArray;
};

export type FixOptions = {
  mode: 'single' | 'spriteSheet' | 'characterSheet' | 'tileSheet';
  targetWidth?: number;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [labcoder/pixel-aid](https://github.com/labcoder/pixel-aid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
