---
trigger: always_on
description: Development instructions and conventions for Vector, the modern SVG editor with filter pipeline and animation system.
---

# Vector - SVG Editor Development Guide

Development instructions and conventions for Vector, the modern SVG editor with filter pipeline and animation system.

## Project Overview

Vector is a Figma-inspired SVG editor that brings professional-grade filter and animation capabilities to web-based vector editing. Key innovations:

1. **Visual Filter Pipeline**: React Flow integration for building complex SVG filter effects
2. **Timeline-Based Animation System**: Native SMIL animation support with keyframe editing  
3. **Unified Node Editor**: Both filters and animations use the same visual pipeline interface

## Tech Stack

- **Runtime**: Bun (fast package manager and runtime)
- **Framework**: React 19 (with concurrent features and new JSX transform)
- **Build Tool**: Vite (fast development and optimized builds)
- **UI Components**: Shadcn/ui (modern, accessible component library)
- **State Management**: Nanostores (atomic, reactive state management)
- **Pipeline Editors**: React Flow (node-based visual editor for filters and animations)
- **Styling**: Tailwind CSS (utility-first CSS framework)
- **Type Safety**: TypeScript (full type coverage)

## Development Commands

```bash
# Install dependencies
bun install

# Start development server
bun dev

# Build for production
bun run build

# Preview production build
bun run preview

# Run type checking
bun run typecheck

# Run linting
bun run lint

# Run linting with auto-fix
bun run lint:fix

# Run tests
bun test

# Run tests in watch mode
bun test --watch
```

## Claude AI Development Guidelines

- Never run the dev server (bun run dev) by yourself. Ask me to test the web app by directing me what to test and expect. You may however run "bun run build" to check if there's any errors in the syntax or build time.
- **Shadcn UI Workflow**: Install shadcn ui components using CLI whenever needed.

## Architecture Decision: SVG-Based Rendering

Vector uses native SVG rendering instead of Canvas for the following reasons:

- **Native Filter Support**: SVG filters (`<defs><filter>`) work natively without conversion or polyfills
- **Native Animation Support**: SMIL animations (`<animate>`, `<animateTransform>`) embedded directly in SVG
- **Vector Precision**: True scalable vectors with no pixelation at any zoom level
- **DOM Integration**: Direct manipulation with React components and native event handlers
- **Export Simplicity**: Direct SVG output with embedded filters and animations
- **Pipeline Integration**: Perfect match with React Flow - generates actual SVG definitions

## Code Conventions

### Component Structure
- Use functional components with hooks
- Follow the established component hierarchy in `/src/components/`
- Implement proper TypeScript types for all props and state
- Use Nanostores for state management, not React state for global data

### State Management
```typescript
// Use Nanostores atoms for reactive state
export const canvasStore = atom({
  shapes: [] as Shape[],
  frames: [] as Frame[],
  viewBox: { x: 0, y: 0, width: 512, height: 512 },
  zoom: 1
});
```

### File Organization
- **Components**: Group by feature (canvas/, panels/, tools/, filters/, animation/)
- **Stores**: One store per domain (canvas.ts, selection.ts, tools.ts, etc.)
- **Types**: Match store structure (canvas.ts, filters.ts, animations.ts, etc.)
- **Utils**: Helper functions organized by purpose

### Styling
- Use Tailwind CSS utility classes
- Follow Shadcn/ui design patterns
- Implement proper dark/light theme support using CSS variables
- Use consistent spacing and color schemes

### TypeScript
- Maintain 100% type coverage
- Define interfaces for all data structures
- Use proper generic types for reusable components
- Export types from dedicated type files

## Key Development Patterns

### React Flow Integration
```typescript
// Filter/Animation nodes should follow this pattern
export function FilterNode({ data, id }: NodeProps) {
  const updateNode = useStore($filterStore, store => store.updateNode);
  
  return (
    <div className="filter-node">
      <div className="node-header">Node Title</div>
      <div className="node-controls">
        {/* Node-specific controls */}
      </div>
      <Handle type="target" position={Position.Left} />
      <Handle type="source" position={Position.Right} />
    </div>
  );
}
```

### SVG Component Pattern
```typescript
// Shape components should render proper SVG elements
export function ShapeComponent({ shape, ...props }: ShapeProps) {
  return (
    <g transform={`translate(${shape.x}, ${shape.y})`}>
      {/* Render actual SVG shape */}
      <rect width={shape.width} height={shape.height} />
      {/* Include animations if present */}
      {shape.animations?.map(animation => (
        <animateTransform
          attributeName="transform"
          type={animation.type}
          values={animation.values}
          dur={animation.duration}
        />
      ))}
    </g>
  );
}
```

### Store Updates
```typescript
// Always use proper store update patterns
const updateShape = (id: string, updates: Partial<Shape>) => {
  canvasStore.set({
    ...canvasStore.get(),
    shapes: canvasStore.get().shapes.map(shape => 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pavi2410/vector](https://github.com/pavi2410/vector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
