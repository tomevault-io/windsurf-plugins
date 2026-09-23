---
trigger: always_on
description: These instructions are for making changes in the LittleJS repo safely. Optimize for small diffs, clarity, and ease of use.
---

# LittleJS Project - AI Agent Instructions

These instructions are for making changes in the LittleJS repo safely. Optimize for small diffs, clarity, and ease of use.

## Non-negotiable rules

- **Prefer minimal, local changes.** Do not refactor for style unless asked.
- **No new runtime dependencies.** Keep LittleJS dependency-free at runtime.
- **Do not hand-edit generated build artifacts.**
  - Treat `dist/` as generated output.
  - Make changes in `src/` (and `plugins/` when appropriate), then run the build.
- **Match surrounding style.** Follow the conventions in the files you touch.
- **Avoid breaking public APIs.** If a change could break users, call it out clearly and offer a compatible alternative.
- **Keep agent-generated working files under `.claude/`.** `docs/` is the published JSDoc API site and is fully generated — never hand-edit it, and don't regenerate it unless asked (see Documentation below). Superpowers plans go in `.claude/superpowers/plans/` and specs in `.claude/superpowers/specs/` (overrides the skill defaults). The `.claude/` folder is gitignored.

If anything in this doc conflicts with the actual repo behavior, follow the repo behavior and update this doc.

## Key resources

- `README.md` - Overview and getting started
- `REFERENCE.md` - API quick reference
- `examples/` - Working examples demonstrating engine features

## Architecture overview

LittleJS is a modular HTML5 game engine with:

- **Core engine**: `src/engine*.js` (main loop, objects, rendering, physics, input, etc.)
- **Plugins**: `plugins/*.js` (optional features like Box2D, post-processing, UI, audio helpers, etc.)
- **3D**: `plugins/math3d.js` (Vector3, Matrix4, Ray3D, 3D collision and raycasts) and `plugins/render3d.js` (the 3D renderer: meshes, builders, lights, shadows, EngineObject3D, cameras, particles). Both are plugins in the same bundle, and `plugins/threejs.js` is the alternative that renders with Three.js.
- **Build system**: `src/engineBuild.mjs` (concatenates modules into distributable bundles)

## Repo structure and file types

### Engine source (`src/*.js`)
- Modular architecture (one subsystem per file)
- Concatenated at build time (internal source does not use ES modules)
- Code is vanilla JavaScript with type info expressed via JSDoc comments

### Build output (`dist/`)
Common outputs include:
- `littlejs.js` - Full bundle (debug features included)
- `littlejs.release.js` - Production bundle (debug stripped)
- `littlejs.esm.js` - ES module build (import/export)
- `littlejs.esm.min.js` - Minified ES module
- `littlejs.d.ts` - TypeScript definitions

Use via script tag or ES module import:
- `<script src="dist/littlejs.js"></script>`
- `import * as LJS from './dist/littlejs.esm.js'`

Prefer adding new optional features as plugins when it keeps the core simpler.

### Examples
- `examples/starter/` - Plain JavaScript global usage via `<script>` (recommended starting point)
- `examples/module/` - ES module import pattern
- `examples/typescript/` - TypeScript example usage
- `examples/shorts/*.js` - Single-file demos loaded by the shorts harness; the 3D ones are `render3d*.js`
- `examples/3d/` - The 3D plugin in one scene; `examples/threejs/` - the same idea rendered with Three.js

### Short examples (`examples/shorts/*.js`)
Short examples are special:
- Pure JS code file, no HTML
- No imports, do not use LJS namespace - engine APIs are available globally
- Override hooks: `gameInit()`, `gameUpdate()`, `gameUpdatePost()`, `gameRender()`, `gameRenderPost()`
- Lines stay within 80 columns, and colors are written with `hsl(...)`, never `rgb(...)`
- Each short is listed in `examples/shorts.js` with a name, a short description and search keywords

## Coding conventions

### Factory functions vs constructors
Prefer factory functions for core types:
- `vec2(x, y)` not `new Vector2(x, y)`
- `rgb(r, g, b, a)` or `hsl(h, s, l, a)` not `new Color(...)`
- `tile(index, size)` for tile info

Use constructors for game objects and complex types:
- `new EngineObject(pos, size)`
- `new ParticleEmitter(...)`
- `new Sound(zzfxParams)`
- `new Timer(duration)`

### Naming
- `camelCase` for variables and functions
- `PascalCase` for classes
- `UPPER_CASE` for constants that are truly constant (like `PI`)

### Code style
- Use JSDoc with `@memberof` grouping (namespaces: Engine, Math, Draw, Input, Audio, Debug, Settings, etc.)
- Prefer single-line comments: `// comment`
- Use `ASSERT(condition, 'error message')` for validation (stripped in release)
- Use `LOG(...)` for debug output (stripped in release)

### Type checking
Use built-in type helpers for validation:
```javascript
isNumber(n)   // true if number and not NaN
isStringLike(s) // true if stringifiable (has toString returning a string)
isArray(a)    // true if array
isVector2(v)  // true if valid Vector2
isVector3(v)  // true if valid Vector3 (3D math plugin)
isColor(c)    // true if valid Color
```

A field that starts as `undefined` reaches `dist/littlejs.d.ts` as `any`, because tsc infers field types from their assignments. Give it a `@type` tag beside its `@property` tag, the way `fogColor` and `obj.shader` do:
```javascript
/** @property {Shader|undefined} - Custom shader to render with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KilledByAPixel/LittleJS](https://github.com/KilledByAPixel/LittleJS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
