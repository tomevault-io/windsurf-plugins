---
trigger: always_on
description: Programmatic video rendering framework using React/TypeScript. Write React components as video compositions, preview them in a player or studio, and render to MP4/WebM via headless Chromium + FFmpeg.
---

# Rendiv

Programmatic video rendering framework using React/TypeScript. Write React components as video compositions, preview them in a player or studio, and render to MP4/WebM via headless Chromium + FFmpeg.

## Project Structure

Monorepo: pnpm workspaces + Turborepo.

| Package | npm name | Build | Purpose |
|---|---|---|---|
| `packages/rendiv` | `@rendiv/core` | `vite build` (library mode, ESM+CJS) | Core: hooks, components, animation, contexts |
| `packages/player` | `@rendiv/player` | `vite build` (library mode, ESM+CJS) | Browser `<Player>` component |
| `packages/bundler` | `@rendiv/bundler` | `tsc` (ESM only) | Vite-based project bundler |
| `packages/renderer` | `@rendiv/renderer` | `tsc` (ESM only) | Playwright frames + FFmpeg stitching |
| `packages/cli` | `@rendiv/cli` | `tsc` (ESM only) | CLI: `rendiv render/still/compositions/studio` |
| `packages/studio` | `@rendiv/studio` | `tsc` (ESM only) | Studio dev server, Vite plugin, server-side render queue |
| `packages/tsconfig` | `@rendiv/tsconfig` | N/A | Shared TS configs (base, react-library, node-library) |
| `examples/hello-world` | private | vite dev | Demo compositions (HelloWorld, SeriesDemo, ShowcaseDemo) |

### Dependency graph
```
cli → bundler → (vite)
    → renderer → (playwright, ffmpeg)
    → studio → bundler, renderer
player → @rendiv/core (peer)
examples → @rendiv/core, player, cli
```

## Commands

```bash
pnpm build        # Build all packages (turbo, dependency-ordered)
pnpm test         # Run all tests (76 tests in packages/rendiv)
pnpm typecheck    # Type-check all packages
pnpm clean        # Remove dist/ from all packages
```

## Code Conventions

### File naming
- All files: `kebab-case` (`use-frame.ts`, `render-frames.ts`)
- Components: `.tsx`, hooks/contexts/utilities: `.ts`

### Exports
- Named exports only, no default exports
- Props interfaces: `<Component>Props`, exported alongside the component
- Types co-exported with their values: `export { Foo, type FooValue }`
- Barrel `src/index.ts` per package, organized by category with comments

### Package exports in package.json
- `"types"` must come first in the exports map (before `"import"` and `"require"`)
- React packages: dual ESM (`.js`) + CJS (`.cjs`) via Vite library mode
- Node packages: ESM only via `tsc`

### Components
- Function components with named export
- `forwardRef` when imperative ref is needed (e.g., `Fill`, `Player`)
- `<Composition>` and `<Still>` render `null` — registration-only via `useEffect` into `CompositionManagerContext`
- Compound component pattern: `Series` + `Series.Sequence` (marker child, throws if rendered directly)

### Contexts
- Live in `packages/rendiv/src/context/` as `.ts` files
- Pattern: `createContext<TypeValue>(default)` + exported interface
- Six contexts: `TimelineContext`, `CompositionContext`, `SequenceContext`, `RendivEnvironmentContext`, `CompositionManagerContext`, `CanvasElementContext`
- Components like `Freeze` and `Loop` override `TimelineContext` (and optionally `SequenceContext`) to alter the frame seen by children
- `<CanvasElement id="...">` provides `CanvasElementContext` to scope override namePaths — always wrap composition content with it so overrides work when nested inside other compositions

### Hooks
- Live in `packages/rendiv/src/hooks/`, prefixed `use-`
- Named function exports: `export function useFrame(): number`

### TypeScript
- Shared configs in `packages/tsconfig/`: `base.json`, `react-library.json`, `node-library.json`
- Target: ES2022, module: ESNext, moduleResolution: bundler, strict: true
- Per-package tsconfig extends the shared preset and sets `outDir: "dist"`, `rootDir: "src"`, `include: ["src"]`

## Public API Names

Key exports from `@rendiv/core`:

| Category | Names |
|---|---|
| Hooks | `useFrame`, `useCompositionConfig` |
| Components | `Composition`, `Sequence`, `Fill`, `Still`, `Folder`, `CanvasElement` |
| Core Components | `Series`, `Series.Sequence`, `Loop`, `Freeze` |
| Media Components | `Img`, `Video`, `Audio`, `AnimatedImage`, `IFrame` |
| Animation | `interpolate`, `spring`, `getSpringDuration`, `Easing`, `blendColors` |
| Registration | `setRootComponent`, `getRootComponent` |
| Render control | `holdRender`, `releaseRender`, `abortRender`, `getPendingHoldCount`, `getPendingHoldLabels` |
| Types | `CompositionConfig`, `CompositionEntry`, `SpringConfig`, `ResolveConfigFunction`, `HoldRenderOptions` |
| Context fields | `accumulatedOffset`, `localOffset`, `parentOffset`, `playingRef` |

## Architecture

### Frame-driven rendering
Video = function of frame number. `useFrame()` returns the current frame, offset by any `<Sequence>` nesting via `SequenceContext`.

### Context override pattern
- `<Freeze frame={N}>` overrides `TimelineContext.frame` to freeze children at frame N
- `<Loop durationInFrames={N}>` overrides both `TimelineContext` and `SequenceContext` using modulo arithmetic so children see frames in `[0, N)`
- `<Series>` delegates to `<Sequence>` with auto-calculated `from` offsets

### holdRender pattern

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thecodacus/rendiv](https://github.com/thecodacus/rendiv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
