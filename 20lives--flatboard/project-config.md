---
trigger: always_on
description: TypeScript-based parametric design system for generating 3D-printable keyboard cases (split, unibody, macropads). Uses scad-js to transpile TypeScript to OpenSCAD, enabling programmatic CAD modeling with modern language features.
---

# flatboard: Parametric Keyboard Case Generator

## Project Overview

TypeScript-based parametric design system for generating 3D-printable keyboard cases (split, unibody, macropads). Uses scad-js to transpile TypeScript to OpenSCAD, enabling programmatic CAD modeling with modern language features.

## Core Technology Stack

- **TypeScript**: Configuration and geometry logic
- **scad-js v0.6.9**: TypeScript-to-OpenSCAD transpiler (also renders STL directly)
- **fp-ts**: Functional programming patterns (pipe, Option, Either, Array utilities)
- **Bun**: Runtime and build system with Bun.Glob for profile discovery
- **Biome**: Linting and formatting (120 line width, single quotes, trailing commas)

## Project Structure

```
src/
├── index.ts                  # CLI with object literal command routing
├── build.ts                  # Build orchestration with 3 modes
├── config.ts                 # Configuration factory with explicit construction
├── profile-loader.ts         # Bridge between profiles/ and src/
├── interfaces.ts             # TypeScript type definitions (250 lines)
├── switches.ts               # Switch specifications (Choc, MX)
├── connector-specs.ts        # Connector definitions (USB-C, TRRS, power button)
├── layout.ts                 # Layout geometry calculations (split + unibody mirroring)
├── switch-sockets.ts         # Switch cutout generation
├── connector.ts              # Connector system with face mapping
├── top.ts                    # Top plate assembly (fp-ts pipe)
├── top-mcu-pocket.ts         # MCU pocket with pin access and USB port cutout
├── bottom.ts                 # Bottom case assembly (fp-ts pipe, 8-step pipeline)
├── bottom-pads-sockets.ts    # Silicon pad socket structures (pure functional)
├── bottom-magsafe-ring.ts    # MagSafe ring for magnetic tenting
├── bottom-patterns.ts        # Bottom patterns (honeycomb, circles, square)
├── switch-visualization.ts   # Switch and keycap visualization for preview
├── utils.ts                  # Core utilities (constants, anchor positioning, deepMerge, rotatePoint, createRoundedSquare)
└── assets/
    ├── cherry-mx.ts          # 3D Cherry MX switch model for visualization
    ├── choc.ts               # Choc keycap model with configurable parameters
    └── keycap-generator.ts   # DSA/XDA keycap generator (layered polygon approach)

profiles/
├── index.ts                  # Auto-discovery with Bun.Glob
├── split-36.ts               # 36-key split (MX, MCU pocket, magsafe, honeycomb pattern)
├── corne.ts                  # 42-key split (MX, USB-C, circles pattern, magsafe)
├── sweep.ts                  # 34-key split (Choc, USB-C, honeycomb pattern, magsafe)
├── planck.ts                 # 48-key unibody ortholinear (MX, USB-C)
├── unibody-36.ts             # 36-key unibody (Choc, USB-C, power button)
├── macropad-3x3.ts           # 9-key macropad (MX, MCU pocket, honeycomb pattern)
├── test-single-choc.ts       # Single Choc key test
└── test-single-mx.ts         # Single MX key test

dist/                         # Generated output
├── <profile>-<hash>/         # Production builds (timestamped)
│   ├── top.scad / top.stl
│   ├── bottom.scad / bottom.stl
│   └── complete.scad / complete.stl
└── (dev mode writes directly to dist/)
```

## Configuration System

### Profile Separation

Keyboard configurations are separated from the codebase in the `profiles/` directory:

- **User space**: `profiles/` contains keyboard definitions (data only)
- **Code space**: `src/` contains the parametric generator logic
- **Auto-discovery**: `profiles/index.ts` uses Bun.Glob to scan and dynamically import all profile files

```typescript
// profiles/index.ts
const glob = new Glob('*.ts');
const profileFiles = Array.from(glob.scanSync({ cwd: __dirname }))
  .filter((file) => file !== 'index.ts');

for (const file of profileFiles) {
  const filePath = join(__dirname, file);
  const module = await import(filePath);
  const profileName = file.replace(/\.ts$/, '');

  if (module.profile) {
    profileEntries.push([profileName, module.profile]);
  }
}
```

### Configuration Flow

```
profiles/*.ts → PROFILES → profile-loader.ts → config.ts → SWITCH_SPECS → KeyboardConfig
```

1. Profile files export a `profile` constant of type `ParameterProfile`
2. `profiles/index.ts` dynamically imports all profiles using Bun.Glob
3. `profile-loader.ts` provides type-safe access to profiles (thin bridge: `getProfileNames()`, `profileExists()`, `getProfile()`)
4. `config.ts` constructs `KeyboardConfig` explicitly from profile + switch specs, using fp-ts Option for null-safe profile lookup
5. `normalizeEdgeMargin()` converts `number | EdgeMargin | undefined` → `EdgeMargin` object
6. Switch spec `spacingX`/`spacingY` used as defaults if not set in profile
7. Final `KeyboardConfig` contains all required parameters with proper defaults

### Row Layout System

Each row defined by parameters:
```typescript
rowLayout: [
  { start: 0, length: 3, offset: 5, thumbAnchor: 2 },  // Row 0: 3 keys, 5mm stagger, thumb anchored to key 2

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [20lives/flatboard](https://github.com/20lives/flatboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
