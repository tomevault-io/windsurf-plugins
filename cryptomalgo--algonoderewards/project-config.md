---
trigger: always_on
description: - Never create README or documentation files unless specifically asked
---

# AlgoNodeRewards - GitHub Copilot Instructions

## General Guidelines

### Code Organization

- Never create README or documentation files unless specifically asked
- Split large files into smaller, focused modules
- Break down complex files into separate files with specific functions to improve readability and maintainability
- Keep functions small and single-purpose
- Extract reusable logic into separate utilities or hooks
- Do not add comment to everything, explain it in the Chat but only add comments in the code where necessary for clarity (complex logic, important notes)
- Do not add JSDoc comments unless specifically requested (no @param or @returns etc, we use TypeScript)
- When you edit test files, run tests using VSCode test explorer instead of the terminal.
- Do not create unused function that might be useful later, only implement what is needed for the current task
- When you spend some time understanding the code, add a brief summary on this file (./.github/copilot-instructions.md) on the most suitable section

### Development Practices

- Implement changes directly rather than just suggesting them
- Write tests for new functionality
- Ensure TypeScript types are properly defined
- Follow existing code patterns and conventions in the codebase
- **Always run `npm run ci` after big changes** to check for errors, lint files, type-check, build, and run tests

### Loading States

- **Use Skeleton components for loading states** - NOT Spinner components
- Skeleton component is located at `src/components/ui/skeleton.tsx`
- Create custom fallback components that match the structure of the content being loaded
- Examples in `src/components/address/address-view.tsx`:
  - `StatsFallback` - Mimics stats panel layout with skeleton placeholders
  - `HeatmapFallback` - Shows skeleton grid matching heatmap structure
  - `ChartFallback` - Displays skeleton for chart containers
- Use `<Skeleton className="h-4 w-32" />` with specific height/width classes
- Wrap skeletons in the same container structure as actual content for seamless transitions

### Communication

- Keep responses concise and to the point
- Focus on implementation over explanation
- Only provide detailed explanations when explicitly requested

---

## Project Overview

AlgoNodeRewards is a React-based web application for tracking and visualizing rewards from running an Algorand node. It provides comprehensive statistics, charts, and analytics for node operators using the Nodely API.

**Website**: [algonoderewards.com](https://algonoderewards.com)  
**Repository**: github.com/cryptomalgo/algonoderewards  
**License**: MIT

## Tech Stack

### Core Framework & Build Tools

- **React 19.2.0** - UI framework
- **TypeScript 5.9.3** - Type-safe JavaScript
- **Vite 7.1.10** - Build tool and dev server
- **Vitest 3.2.4** - Unit testing framework

### Routing & State Management

- **TanStack Router 1.133.3** - File-based routing with type safety
  - Routes are in `src/routes/` folder
  - Auto-generated route tree in `src/routeTree.gen.ts`
  - Uses `createFileRoute` for route components
- **TanStack Query 5.90.5** - Server state management and data fetching
  - Queries are organized in `src/hooks/` and `src/queries/`

### UI & Styling

- **Tailwind CSS 4.1.14** - Utility-first CSS framework
  - Configuration in `@tailwindcss/vite` plugin
  - Custom CSS in `src/App.css` with CSS variables for theming
  - Dark/light/system theme support via `next-themes`
- **shadcn/ui** - Component library (New York style)
  - Components in `src/components/ui/`
  - Configuration in `components.json`
  - Uses Radix UI primitives
  - Lucide React for icons
  - `class-variance-authority` for component variants
  - `tailwind-merge` and `clsx` for className composition

### Radix UI Components

- Dialog, Dropdown Menu, Checkbox, Label, Popover, Select, Slider, Toggle, Toggle Group, Tooltip

### Data Visualization

- **Recharts 3.3.0** - Charts and graphs
- **@uiw/react-heat-map 2.3.3** - Heatmap visualization
- **react-gauge-component 1.2.64** - Gauge displays

### Algorand Integration

- **algosdk 3.5.2** - Algorand JavaScript SDK
- **@algorandfoundation/algokit-utils 9.1.2** - Algorand utilities
- Uses Nodely API (mainnet-idx.4160.nodely.dev) for indexer access

### Additional Libraries

- **motion 12.23.24** - Animation library (Framer Motion successor)
- **date-fns 4.1.0** - Date manipulation
- **sonner 2.0.7** - Toast notifications
- **react-error-boundary 6.0.0** - Error handling
- **react-day-picker 9.11** - Calendar/date picker

## Project Structure

```
src/
├── routes/              # TanStack Router file-based routes
│   ├── __root.tsx       # Root layout with header, footer, devtools
│   ├── index.tsx        # Landing page with search
│   ├── $addresses.tsx   # Address rewards view (dynamic route)
│   └── privacy-policy.tsx
│
├── components/          # React components
│   ├── ui/              # shadcn/ui components (Button, Dialog, etc.)
│   ├── address/         # Address-specific components
│   │   ├── charts/      # Reward/block charts and visualizations
│   │   └── stats/       # Statistics panels and boxes
│   ├── heatmap/         # Heatmap components
│   └── [other components]
│
├── hooks/               # Custom React hooks & queries

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cryptomalgo/algonoderewards](https://github.com/cryptomalgo/algonoderewards) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
