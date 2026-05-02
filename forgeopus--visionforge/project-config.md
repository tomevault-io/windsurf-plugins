---
trigger: always_on
description: VisionForge is a visual neural network architecture design tool with drag-and-drop block composition, automatic dimension inference, and PyTorch code generation. It's a monorepo with a React/TypeScript frontend and Django backend (currently minimal).
---

# VisionForge AI Model Builder - Copilot Instructions

## Project Overview
VisionForge is a visual neural network architecture design tool with drag-and-drop block composition, automatic dimension inference, and PyTorch code generation. It's a monorepo with a React/TypeScript frontend and Django backend (currently minimal).

**Tech Stack:**
- **Frontend**: React 19, TypeScript, Vite, @xyflow/react (canvas), Zustand (state), Radix UI + Tailwind CSS
- **Backend**: Django 5.2, Python 3.12+ (currently scaffolded but minimal)
- **Key Dependencies**: @github/spark, framer-motion, react-hook-form, zod

## Architecture & Data Flow

### State Management (Zustand)
- **Single store**: `src/lib/store.ts` - all application state lives here
- State structure: `nodes`, `edges`, `selectedNodeId`, `validationErrors`, `currentProject`
- **Key pattern**: State mutations trigger `inferDimensions()` to propagate tensor shapes through the graph
- No Redux, no Context API - Zustand hooks like `useModelBuilderStore()` are the only state access pattern

### Block System
1. **Definitions**: `src/lib/blockDefinitions.ts` - registry of all available neural network blocks
   - Each block has: `type`, `label`, `category`, `color`, `icon`, `configSchema`, `computeOutputShape()`
   - Categories: `input`, `basic`, `advanced`, `merge`
   - **Connection validation**: `validateBlockConnection()` enforces dimension compatibility rules
   - **Multi-input support**: `allowsMultipleInputs()` identifies merge blocks (concat, add)
2. **Type System**: `src/lib/types.ts` - strict TypeScript interfaces for `BlockData`, `TensorShape`, `BlockConfig`
3. **Dimension Inference**: Automatic shape propagation via `inferDimensions()` in store
   - Walks graph from input nodes, computes output shapes using `computeOutputShape()` functions
   - Updates triggered on node addition, edge creation, or config changes
4. **Custom Layers**: Special handling with modal dialog (`CustomLayerModal.tsx`) for code editing
   - Uses CodeMirror (@uiw/react-codemirror) for Python syntax highlighting
   - Configuration shown in modal, not sidebar
   - User can write custom forward pass logic

### Canvas & Flow
- Uses **@xyflow/react** for node-based editor (not React Flow - this is XYFlow)
- Custom node component: `BlockNode.tsx` renders blocks with colored borders, shape annotations
- Connection validation: `validateConnection()` prevents invalid connections (e.g., Conv2D requires 4D input)
- **Pattern**: Canvas operations go through store actions, never direct ReactFlow state manipulation

### Code Generation
- `src/lib/codeGenerator.ts` generates complete PyTorch projects (model.py, train.py, config.py)
- Topological sort for layer ordering, then translate block configs to PyTorch syntax
- Currently PyTorch-only (TensorFlow generation stubbed)

## Development Workflows

### Running the App
```bash
# Frontend (from project/frontend/)
npm run dev          # Vite dev server on port 5173
npm run build        # Production build
npm run lint         # ESLint

# Backend (from project/)
python manage.py runserver  # Django dev server on port 8000 (not currently used)
```

### Adding a New Block Type
1. Add type to `BlockType` union in `types.ts`
2. Create definition in `blockDefinitions.ts` with `computeOutputShape()` logic
3. No UI changes needed - palette and config panel auto-generate from schema

### State Mutations
**Always** use store actions - never mutate nodes/edges directly:
```typescript
// ✅ Correct
updateNode(id, { config: { ...config, param: value } })

// ❌ Wrong
node.data.config.param = value
```

## Project-Specific Conventions

### Styling
- **Triadic color scheme**: Deep Blue (primary), Teal (input/output), Purple (advanced), Cyan (accent)
- Colors defined as CSS custom properties in `styles/theme.css`
- Use `var(--color-primary)`, `var(--color-accent)`, etc. - never hardcode colors
- Tailwind for layout, custom properties for semantic colors

### Component Patterns
- Radix UI primitives wrapped in `components/ui/` (shadcn/ui style)
- Always use `forwardRef` for UI components that accept refs
- Phosphor Icons (`@phosphor-icons/react`) for all icons
- Framer Motion for animations (spring physics, not easing curves)

### Type Safety
- Strict TypeScript - no `any` without explicit justification
- Zod schemas for form validation (see `react-hook-form` + `@hookform/resolvers`)
- Block configs are type-safe via `BlockConfig` interface

### Error Handling
- Validation errors stored in `validationErrors` array with `nodeId`, `message`, `type` fields
- Toast notifications via `sonner` library
- No try-catch without logging - errors should be visible to users

## Integration Points

### API (Currently Unused)
- `src/lib/api.ts` defines API client for backend
- Backend endpoints planned: `/api/validate`, `/api/chat`, `/api/export`
- **Current state**: All operations are client-side; backend is scaffolded but not integrated

### File Structure
```
project/
  frontend/           # React SPA
    src/
      components/     # React components
        ui/          # Radix UI wrappers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ForgeOpus/visionforge](https://github.com/ForgeOpus/visionforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
