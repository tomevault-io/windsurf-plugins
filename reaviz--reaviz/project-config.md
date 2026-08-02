---
trigger: always_on
description: REAVIZ is a modular React charting library that uses D3.js for calculations and React for rendering. It provides a composable API for building data visualizations with features like animations, tooltips, legends, and accessibility support.
---

# CLAUDE.md - AI Assistant Guide for REAVIZ

## Project Overview

REAVIZ is a modular React charting library that uses D3.js for calculations and React for rendering. It provides a composable API for building data visualizations with features like animations, tooltips, legends, and accessibility support.

- **Repository**: https://github.com/reaviz/reaviz
- **License**: Apache-2.0
- **Node Version**: >=22
- **Package Manager**: npm (>=10.8.2)

## Quick Reference Commands

```bash
# Install dependencies
npm install

# Development (Storybook)
npm start                  # Start Storybook dev server on port 9009

# Build
npm run build              # Full build (JS + docs)
npm run build:js           # Build library only (Vite)
npm run build-storybook    # Build Storybook static site

# Testing
npm test                   # Run tests with Vitest (watch mode)
npm run test:ci            # Run tests with coverage
npm run test-storybook     # Run Storybook visual tests

# Linting & Formatting
npm run lint               # Run ESLint
npm run lint:fix           # Run ESLint with auto-fix
npm run prettier           # Format code with Prettier
npm run ts:check           # TypeScript type checking
```

## Project Structure

```
reaviz/
├── src/                    # Source code
│   ├── index.ts            # Main exports
│   ├── common/             # Shared components and utilities
│   │   ├── Axis/           # Linear and Radial axis components
│   │   ├── Brush/          # Brush selection component
│   │   ├── containers/     # Chart container components
│   │   ├── data/           # Data transformation utilities
│   │   ├── Gradient/       # SVG gradient components
│   │   ├── Gridline/       # Grid line components
│   │   ├── legends/        # Legend components
│   │   ├── Mask/           # SVG mask/pattern components
│   │   ├── Motion/         # Animation utilities
│   │   ├── scales/         # D3 scale utilities
│   │   ├── Tooltip/        # Tooltip components
│   │   ├── utils/          # General utilities
│   │   └── ZoomPan/        # Pan/zoom gesture handling
│   │
│   └── [ChartType]/        # Individual chart implementations
│       ├── ChartType.tsx         # Main chart component
│       ├── ChartType.module.css  # Component styles
│       ├── ChartType*.story.tsx  # Storybook stories
│       ├── [SubComponents]/      # Chart-specific sub-components
│       └── index.ts              # Barrel exports
│
├── blocks/                 # Pre-built chart compositions
├── docs/                   # MDX documentation for Storybook
├── test/                   # Test utilities and mocks
├── scripts/                # Build scripts
└── .storybook/             # Storybook configuration
```

## Chart Types Available

- **AreaChart** - Single/multi-series area charts with stacking
- **BarChart** - Vertical/horizontal bars, grouped, stacked, waterfall, Marimekko
- **BarList** - Simple horizontal bar list component
- **BubbleChart** - Bubble/scatter with size dimension
- **FunnelChart** - Funnel visualization
- **Heatmap** - Standard and calendar heatmaps
- **LineChart** - Single/multi-series line charts
- **LinearGauge** - Linear gauge visualization
- **Map** - Geographic map charts
- **Meter** - Simple meter/progress component
- **PieChart** - Pie and donut charts
- **RadarChart** - Spider/radar charts
- **RadialAreaChart** - Radial area charts
- **RadialBarChart** - Radial bar charts
- **RadialGauge** - Radial gauge visualization
- **RadialScatterPlot** - Radial scatter plots
- **Sankey** - Sankey flow diagrams
- **ScatterPlot** - Scatter plot charts
- **Sparkline** - Compact inline charts
- **SunburstChart** - Hierarchical sunburst
- **TreeMap** - Hierarchical treemap
- **VennDiagram** - Venn and Euler diagrams
- **WordCloud** - Word cloud visualization

## Code Conventions

### Component Architecture

1. **Functional Components**: All components use React functional components with hooks
2. **TypeScript**: Full TypeScript with explicit prop interfaces
3. **Props Pattern**: Components define a Props interface (e.g., `BarChartProps`) extending base types
4. **Default Props**: Use `mergeDefaultProps()` utility with a `*_DEFAULT_PROPS` constant

```tsx
// Standard component structure
export interface BarChartProps extends ChartProps {
  data: ChartDataShape[];
  series: ReactElement<BarSeriesProps, typeof BarSeries>;
  // ... other props with JSDoc comments
}

export const BarChart: FC<Partial<BarChartProps>> = (props) => {
  const { data, series, ... } = mergeDefaultProps(BAR_CHART_DEFAULT_PROPS, props);
  // Component implementation
};

const BAR_CHART_DEFAULT_PROPS = {
  data: [],
  series: <BarSeries />,
  // ... defaults
};
```

### Import Conventions

- Use `@/` path alias for `src/` imports (configured in tsconfig.json)
- Prefer named exports over default exports
- Use barrel exports (`index.ts`) for each module

```tsx
import { BarSeries, Bar, BarLabel } from '@/BarChart/BarSeries';
import { ChartContainer } from '@/common/containers/ChartContainer';
```

### Styling

- **CSS Modules**: Use `*.module.css` files for component styles

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reaviz/reaviz](https://github.com/reaviz/reaviz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
