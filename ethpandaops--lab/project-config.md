---
trigger: always_on
description: pnpm dev                    # Dev server proxying to local backend (localhost:8080)
---

# Lab

## Commands

```bash
pnpm dev                    # Dev server proxying to local backend (localhost:8080)
BACKEND=production pnpm dev # Dev server proxying to production (lab.ethpandaops.io)
pnpm lint
pnpm test
pnpm build
pnpm storybook
```

The `BACKEND` env var controls the API proxy target. Values: `local` (default), `production`, or any custom URL.

## Libraries

- pnpm v10, node v24, vite v7, react v19, typescript v5
- tailwindcss v4, headlessui v2, heroicons v2
- @tanstack/react-query v5, @tanstack/router-plugin v1
- zod v4, react-hook-form v7, clsx
- echarts v6, echarts-for-react v3, echarts-gl v2
- storybook v10, vitest v4

## Project Structure

```bash
src/
  routes/                             # Thin TanStack Router definitions
    __root.tsx                        # Root layout with sidebar, providers, navigation
    index.tsx                         # "/" - Landing page route
    [section].tsx                     # Layout routes (ethereum, xatu, experiments)
    [section]/                        # Section-specific routes
      [page-name].tsx                 # Page route
      [page-name]/                    # Nested/sub-section routes
        index.tsx                     # Default nested route
        $param.tsx                    # Dynamic parameter route

  pages/                              # Page components (actual UI implementation)
    home/                             # Landing page
    [section]/                        # Section-specific pages
      [page-name]/                    # Individual page folder
        IndexPage.tsx                 # Main page component
        IndexPage.types.ts            # Search param types, page-specific types
        [OtherPage].tsx               # Additional pages (e.g., DetailPage)
        index.ts                      # Barrel export
        components/                   # Page-specific components
        hooks/                        # Page-specific hooks
        contexts/                     # Page-specific contexts
        providers/                    # Page-specific providers
        utils/                        # Page-specific utilities
        constants.ts                  # Page-specific constants

  components/                         # Core, app-wide reusable UI components
    [Category]/[ComponentName]/       # Category → Component folder
      [ComponentName].tsx             # Implementation
      [ComponentName].test.tsx        # Vitest tests
      [ComponentName].types.ts        # TypeScript types (optional)
      [ComponentName].stories.tsx     # Storybook stories
      index.ts                        # Barrel export

  providers/                          # React Context Providers
  contexts/                           # React Context definitions (with .types.ts)
  hooks/                              # Custom React hooks (with tests)
  api/                                # Generated API client (do not edit)
    @tanstack/react-query.gen.ts      # TanStack Query hooks - USE THIS for all API calls
  utils/                              # Utility functions and helpers
  index.css                           # Global styles and Tailwind theme config
```

## Architecture

### Component & State Philosophy

**Core/Shared** (`src/components/`, `src/hooks/`, `src/contexts/`):
- App-wide reusable building blocks — generic and configurable
- Work in any context without page logic
- Core components include Storybook stories

**Page-Scoped** (`src/pages/[section]/[page-name]/components|hooks|contexts|providers|utils/`):
- Used within specific page/section only
- Compose/extend core items for page needs
- Contain page-specific business logic

**Placement rule:** Used across pages → Core. Page-specific → Page-scoped.

### Core Component Categories

- **Charts**: Bar, BoxPlot, Donut, FlameGraph, GasFlowDiagram, Gauge, Globe, GridHeatmap, Heatmap, Line, Map, Map2D, MultiLine, Radar, ScatterAndLine, Sparkline, StackedBar
- **DataDisplay**: CardChain, GasTooltip, MiniStat, Stats, Timestamp
- **DataTable**: DataTable (compound component with sub-components)
- **DateTimePickers**: DatePicker
- **Elements**: Avatar, Badge, Button, ButtonGroup, CopyToClipboard, Dropdown, Icons, TimezoneToggle
- **Ethereum**: ~32 blockchain-specific components (ClientLogo, Entity, Epoch, ForkLabel, NetworkIcon, NetworkSelect, Slot, SlotTimeline, etc.)
- **Feedback**: Alert, InfoBox
- **Forms**: Checkbox, CheckboxGroup, Input, RadioGroup, RangeInput, SelectMenu, TagInput, Toggle
- **Layout**: Card, Container, Disclosure, Divider, Header, ListContainer, LoadingContainer, PopoutCard, ScrollArea, Sidebar, ThemeToggle
- **Lists**: ScrollingTimeline, Table
- **Navigation**: Breadcrumb, ProgressBar, ProgressSteps, ScrollableTabs, ScrollAnchor, Tab
- **Overlays**: ConfigGate, Dialog, FatalError, FeatureGate, NotFound, Notification, Popover

### Page Sections

- **home**: Landing page
- **ethereum**: Blockchain visualizations (sub-sections: consensus, contracts, data-availability, entities, epochs, execution, forks, slots, validators)
- **xatu**: Xatu data, metrics, contributor insights
- **experiments**: Legacy experiments index

Route directories also include `beacon/` and `xatu-data/` alongside the sections above.

## Development Guidelines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ethpandaops/lab](https://github.com/ethpandaops/lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
