---
trigger: always_on
description: Canonical instruction file for this repository, shared by all coding agents
---

# AGENTS.md

Canonical instruction file for this repository, shared by all coding agents
(Claude Code, Codex, Cursor, and others). Claude Code reaches it through the
`@AGENTS.md` import in `CLAUDE.md`. Repository guidance belongs here, while
tool-specific pointer files contain no duplicate guidance.

## Project knowledge

<!--
Fill this in as the project develops. Keep it under a page so stale context does
not burden every session. When an agent makes the same mistake twice, record the
correction here.
-->

### Project Overview

Vue 3 Carousel is a modern, lightweight carousel component library for Vue 3. It's distributed as a reusable component library with TypeScript support, offering responsive breakpoints, infinite scrolling, touch/mouse drag, keyboard navigation, and accessibility features.

### Development Commands

#### Build and Development
- `pnpm build` - Build the library for production (generates UMD, ESM, and CJS formats in `dist/`)
- `pnpm dev` - Start the Vite development server with the playground app
- `pnpm typecheck` - Run TypeScript type checking
- `pnpm typecheck:watch` - Run TypeScript type checking in watch mode

#### Testing
- `pnpm test` - Run all tests with Vitest and generate coverage report
- `pnpm test:watch` - Run tests in watch mode
- `pnpm test:update` - Update test snapshots

Running a subset (the `test` script always adds `--coverage`, so call `vitest` directly):
- `pnpm vitest run src/utils/throttle.spec.ts` - a single file
- `pnpm vitest run tests/integration` - a single directory
- `pnpm vitest run -t 'should throttle'` - tests matching a name
- `pnpm vitest watch src/components/Carousel` - watch one path

#### Code Quality
- `pnpm lint` - Lint the codebase with ESLint
- `pnpm lint:fix` - Auto-fix ESLint issues
- `pnpm prettier` - Check code formatting
- `pnpm prettier:fix` - Auto-format code with Prettier

#### Documentation
- `pnpm docs:dev` - Start VitePress documentation site in development mode
- `pnpm docs:build` - Build documentation site for production
- `pnpm docs:serve` - Preview built documentation site

#### Release
- `pnpm release` - Runs `scripts/new-release.sh`

### Architecture

#### Component Structure

The library follows a provider/inject pattern for component communication:

1. **Carousel Component** (`src/components/Carousel/Carousel.ts`) - The main container component that:
   - Manages slide state and registry via `createSlideRegistry()`
   - Provides configuration and state to child components via Vue's `provide()` using the `injectCarousel` symbol
   - Handles all core logic: sliding, dragging, autoplay, breakpoints, and animations
   - Exposes navigation methods: `next()`, `prev()`, `slideTo()`
   - Uses composables for drag (`useDrag`), wheel (`useWheel`), and hover (`useHover`) interactions

2. **Slide Component** (`src/components/Slide/Slide.ts`) - Individual slide wrapper that:
   - Injects carousel context via `inject(injectCarousel)`
   - Registers itself with the carousel's slide registry on mount
   - Computes its visibility, active state, and dimensions based on carousel state
   - Prevents nested slides by providing `undefined` for `injectCarousel`

3. **Navigation Component** - Optional prev/next buttons
4. **Pagination Component** - Optional page indicators
5. **ARIA Component** - Accessibility live region announcements

#### Slide Registry Pattern

The slide registry (`src/shared/slideRegistry.ts`) is a reactive system that:
- Maintains a shallow reactive array of slide component instances
- Handles registration/unregistration of slides dynamically
- Updates slide indexes when slides are added/removed
- Emits `slide-registered` and `slide-unregistered` events
- Filters out cloned slides (used for wrap-around effect) from the registry

#### Path Aliases

The codebase uses `@/*` path aliases that map to `src/*`. This is configured in:
- `tsconfig.json` for TypeScript
- `vitest.config.ts` for tests (dynamically resolved from tsconfig paths)
- Rollup for the build process

#### Clone Slides for Wrap-Around

When `wrapAround` is enabled, the carousel creates clone slides before and after the real slides to create the infinite loop effect. The number of clones is calculated dynamically based on `itemsToShow` and the current slide position. Clones are marked with `isClone: true` and excluded from the registry.

#### Reactive Configuration with Breakpoints

The carousel uses a `shallowReactive` config object that merges:
1. `DEFAULT_CONFIG` (from `src/shared/constants.ts`)
2. Component props (excluding `breakpoints` and `modelValue` to avoid unnecessary reactivity)
3. Breakpoint-specific overrides based on viewport or carousel width

The `updateBreakpointsConfig()` function recalculates config when the window/carousel is resized.

#### Transform and Positioning

The carousel track is positioned using CSS transforms:
- Calculates `scrolledOffset` based on current slide index and snap alignment
- Adds `dragOffset` from user interaction
- Applies `clonedSlidesOffset` when wrap-around is active
- Supports both horizontal (translateX) and vertical (translateY) modes
- RTL and bottom-to-top directions are handled via the `normalizedDir` computed property

#### Slide Effects


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ismail9k/vue3-carousel](https://github.com/ismail9k/vue3-carousel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
