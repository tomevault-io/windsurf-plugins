---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Webflow Code Components** library - a collection of interactive React components designed for import into Webflow Designer. The components run in isolated Shadow DOM environments and include visual effects like particle systems, animated grids, and interactive UI elements.

## Development Commands

### Core Commands
```bash
# Start development server (component showcase)
npm start

# Build production bundle
npm run build

# Run tests
npm test

# Share components with Webflow workspace
npx webflow library share
```

### Webflow Authentication
- **Interactive (recommended)**: Run `npx webflow library share` - CLI will open browser for auth
- **Manual**: Set `WEBFLOW_WORKSPACE_API_TOKEN` in `.env` file (must be Workspace Admin)

## Architecture

### Component Structure

Each component follows this pattern:
```
src/components/ComponentName/
├── ComponentName.tsx           # Main React component implementation
├── ComponentName.webflow.tsx   # Webflow declaration & prop mapping
├── ComponentName.css           # Component styles (Shadow DOM)
└── README.md                   # Component documentation
```

### Key Architectural Patterns

1. **Shadow DOM Isolation**: Each component renders in its own Shadow DOM with separate React root
   - Components cannot share React Context or global state
   - Styles are isolated (site classes don't apply)
   - Must explicitly import all CSS
   - Can use CSS variables from `:root` scope (they inherit into Shadow DOM)

2. **Dual Component Pattern**:
   - `ComponentName.tsx` - Pure React implementation for local development
   - `ComponentName.webflow.tsx` - Webflow wrapper that declares props using `@webflow/data-types`

3. **SSR Consideration**: Most components disable SSR (`ssr: false`) because they use:
   - Canvas/WebGL rendering (DotGrid, Particles)
   - Browser-only APIs (window, document)
   - Mouse/pointer event tracking

### Component Communication

Since Shadow DOM isolates components:
- **No React Context** between components
- Use URL parameters, localStorage, or external state libraries (Nano Stores) for shared state
- See `.claude/skills/webflow-code-components/references/component-communication.md` for patterns

### Props and Data Types

Use `@webflow/data-types` for Webflow prop declarations:
- `props.Text()` - Single line text input
- `props.RichText()` - Multi-line text with formatting
- `props.TextNode()` - Text editable directly on canvas
- `props.Number()` - Numeric input with min/max/decimals
- `props.Boolean()` - True/false toggle
- `props.Variant()` - Dropdown with predefined options
- `props.Image()` - Image upload/selection (returns URL string)
- `props.Link()` - URL input (returns object with href/target/preload)
- `props.Visibility()` - Show/hide controls for conditional rendering
- `props.Slot()` - Container for child elements (FAQ component uses this)
- `props.Color()` - Color picker **(NOT YET AVAILABLE)**
- `props.ID()` - HTML element ID for anchors/accessibility **(NOT YET AVAILABLE)**

**Common configuration options:**
- `name` - Display name in Webflow Designer (required)
- `defaultValue` - Default value for the prop
- `group` - Group related props together (e.g., 'Content', 'Style', 'Behavior')
- `tooltip` - Help text shown in Designer
- `min`/`max`/`decimals` - For Number type

**Important**: Props from Webflow may arrive as strings - parse them in the wrapper component (see FAQ.webflow.tsx for example).

## Component Categories

### Interactive Visual Effects
- **DotGrid** - Physics-based dot grid with mouse interaction (Canvas + GSAP)
- **Particles** - 3D particle system with WebGL (OGL library)
- **GridMotion** - Animated grid with gradient effects (GSAP)
- **MagnetLines** - Magnetic line grid that responds to mouse (CSS Grid + GSAP)

### UI Components
- **FAQ** - Accessible accordion component with Collection List support
  - Uses `props.Slot()` to accept Webflow Collection Lists
  - Extracts items via data attributes (`data-faq-question`, `data-faq-answer`)
  - Full keyboard navigation and ARIA support

## Styling Approach

1. **CSS Modules or standalone CSS** - Imported in `.webflow.tsx` file
2. **CSS Variables** - Define in Webflow Variables tool, inherit into Shadow DOM
   - Example: FAQ uses `--faq-question-color`, `--faq-answer-padding`, etc.
3. **No global styles** - Page styles don't affect components
4. **No site classes** - Webflow classes unavailable inside Shadow DOM

## Key Dependencies

- `react` & `react-dom` (v19.2.0) - Component framework
- `gsap` (v3.13.0) - Animation library (used by DotGrid, GridMotion, MagnetLines)
- `ogl` (v1.0.11) - WebGL library (used by Particles)
- `@webflow/react` - Webflow React utilities
- `@webflow/data-types` - Webflow prop type definitions
- `@webflow/webflow-cli` - CLI for publishing to Webflow

## Testing Setup

- Testing library: `@testing-library/react` + `@testing-library/jest-dom`
- Run with: `npm test`
- Setup file: `src/setupTests.js`

## Important Configuration Files

- **webflow.json** - Defines library name and component glob pattern

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zehfred/webflow-code-components](https://github.com/zehfred/webflow-code-components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
