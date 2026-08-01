---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workflow

- Enter plan mode for non-trivial tasks (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan — don't keep pushing
- Use subagents to keep main context clean; one task per subagent
- After corrections: update `tasks/lessons.md` with the pattern
- Never mark a task complete without proving it works (run tests, check logs)
- Autonomous bug fixing: just fix it, don't ask for hand-holding

## Task Management

1. Write plan to `tasks/todo.md` with checkable items
2. Check in before starting implementation
3. Mark items complete as you go; capture lessons in `tasks/lessons.md`

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Minimal code impact.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Only touch what's necessary. Avoid introducing bugs.

## Overview

**Vue Charts (vccs)** — An unofficial Vue 3 port of [Recharts](https://recharts.org/). Composable charting components built with Vue 3 Composition API + JSX/TSX.

- When porting, refer to React source for behavior parity
- Monorepo: `vccs` (library) + `play` (Nuxt playground) + `docs` (Nuxt docs site), managed by pnpm workspaces

## Build & Development Commands

```bash
pnpm install              # Install dependencies
pnpm dev                  # Watch mode
pnpm build                # Build library (alias for --filter vccs build)
pnpm test                 # Run tests
pnpm test:coverage        # Tests with coverage
pnpm storybook            # Storybook
pnpm play                 # Playground
pnpm docs                 # Docs site (Nuxt 3, port 3001)
pnpm pub:release          # Publish

# Run specific test
pnpm test packages/vue/src/chart/__tests__/AreaChart.spec.tsx
```

**CI** (`.github/workflows/test.yml`): triggers on PRs to `main`; runs `pnpm install --frozen-lockfile` → `pnpm --filter vccs build` → `pnpm test` on Node 20 / ubuntu-latest.

## Architecture

```
packages/vue/src/           # Main library source (vccs)
├── cartesian/              # Area, Bar, Line, Scatter, Axis, Brush, CartesianGrid, ZAxis; funnel/ subdirectory
├── polar/                  # Pie, Radar, RadialBar, PolarGrid, PolarAngleAxis, PolarRadiusAxis
├── chart/                  # Chart containers (AreaChart, BarChart, LineChart, ComposedChart, FunnelChart, etc.)
├── components/             # Legend, Tooltip, Text, Label, Cell
├── container/              # ResponsiveContainer, Surface, Layer
├── shape/                  # Rectangle, Symbols, Dot, Sector, Cross, Curve, Trapezoid
├── state/                  # Redux store, slices, middleware, selectors
├── animation/              # Animate component, motion-v utilities
├── context/                # provide/inject context providers
├── hooks/                  # Shared composition hooks
├── storybook/              # Stories
├── types/                  # Shared type definitions
├── utils/                  # Utility functions
└── index.ts                # Public API

docs/                       # Documentation site (Nuxt 3, Docus framework)
playground/nuxt/            # Nuxt 3 playground (Tailwind v4, shadcn-nuxt)
```

### Key Decisions

1. **Components**: `defineComponent` + JSX (not SFC)
2. **State**: Redux Toolkit via `@reduxjs/vue-redux` — one store per chart (`createRechartsStore`)
3. **Context**: `provide/inject` for parent-child communication
4. **Chart Factory**: `generateCategoricalChart()` creates chart containers
5. **Animation**: `motion-v` with `Animate` wrapper
6. **Events**: Redux middleware — mouse events use `createListenerMiddleware`; external/keyboard/touch use plain synchronous `Middleware`
7. **Build output**: ESM-only (`preserveModules: true`); `minify: false` — Rolldown's minifier renames variables colliding with Vue's `h`

## Code Conventions

### Naming
- Components: PascalCase; Directories: kebab-case; Hooks: `use` prefix; Types: `Props` suffix
- Type files: `type.ts`; Tests: `__tests__/*.spec.tsx`; Stories: `__stories__/*.stories.tsx`

### Component Pattern
```typescript
export const Component = defineComponent<PropsWithSVG>({
  name: 'Component',
  props: ComponentVueProps,
  inheritAttrs: false,
  slots: Object as SlotsType<Slots>,
  setup(props, { attrs, slots }) {
    useSetupGraphicalItem(props, 'itemType')
    const { ...data } = useComponentHook(props, attrs)
    return () => (/* JSX */)
  },
})
```

**Volar slot type preservation** (for compiled `.d.ts`):
```typescript
const _Component = defineComponent<PropsWithSVG>({ /* ... */ })
export const Component = _Component as typeof _Component & {
  new (): { $slots: ComponentSlots }
}
```

### Props Pattern
```typescript
export const ComponentVueProps = {
  dataKey: { type: [String, Number, Function] as PropType<DataKey<any>>, required: true },
  fill: { type: String, default: undefined },
}
export type ComponentPropsWithSVG = WithSVGProps<VuePropsToType<typeof ComponentVueProps>>
```

### Imports
- `@/` → `packages/vue/src/`
- Prefer `import type` for type-only imports

## Key Patterns

### Porting from Recharts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unovue/vue-charts](https://github.com/unovue/vue-charts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
