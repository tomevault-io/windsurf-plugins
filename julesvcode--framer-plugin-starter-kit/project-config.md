---
trigger: always_on
description: This is a Framer plugin starter kit using React, TypeScript, Vite, and Tailwind CSS.
---

# Framer Plugin Development Rules

## Project Overview
This is a Framer plugin starter kit using React, TypeScript, Vite, and Tailwind CSS.

## Tech Stack
- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite 6
- **Styling**: Tailwind CSS v3
- **Plugin SDK**: framer-plugin v3
- **Dev Server**: HTTPS via vite-plugin-mkcert (required for Framer)

## Important Configuration Notes

### PostCSS & Tailwind Setup
- Use `postcss.config.js` and `tailwind.config.js` with ES module syntax (`export default`)
- These work with `"type": "module"` in package.json
- Always import CSS at the **main entry point** (`src/main.tsx`), not in components
- Port 5173 must be available - kill any conflicting processes before starting dev server

### Development Server
- Run: `npm run dev`
- Server MUST run on port 5173 (default Vite port) with HTTPS
- If port is in use, kill the process: `lsof -ti:5173 | xargs kill -9`
- Clear Vite cache if issues: `rm -rf node_modules/.vite`

## Framer Plugin API

### Core API (`framer` object)

#### UI Configuration
```typescript
import { framer } from "framer-plugin"

// Show plugin UI (call this at the top level, not in a component)
framer.showUI({
    position: "top right" | "top left" | "bottom right" | "bottom left" | "center",
    width: number,
    height: number,
    resizable?: boolean,
    minWidth?: number,
    minHeight?: number,
    maxWidth?: number,
    maxHeight?: number,
})

// Hide the plugin UI
framer.hideUI()
```

#### Selection Management
```typescript
import { CanvasNode } from "framer-plugin"

// Subscribe to selection changes (returns cleanup function)
const unsubscribe = framer.subscribeToSelection((selection: CanvasNode[]) => {
    console.log("Selection changed:", selection)
})

// Get current selection once
const selection = await framer.getSelection()
```

#### Canvas Manipulation
```typescript
// Add SVG to canvas
await framer.addSVG({
    svg: string,           // SVG markup
    name?: string,         // Layer name in Framer
    x?: number,            // Position
    y?: number,
    width?: number,        // Dimensions
    height?: number,
    fill?: string,         // Override fill color
})

// Add image to canvas
await framer.addImage({
    image: string | Uint8Array,  // Data URL or binary data
    name?: string,
    x?: number,
    y?: number,
    width?: number,
    height?: number,
})

// Add component instance
await framer.addComponentInstance({
    url: string,           // Component URL
    name?: string,
    attributes?: Record<string, any>,
    x?: number,
    y?: number,
})

// Clone nodes
await framer.cloneNode(node: CanvasNode)
```

#### Node Operations
```typescript
// Get node by ID
const node = await framer.getNode(id: string)

// Update node properties
await framer.setAttributes(node: CanvasNode, attributes: {
    name?: string,
    x?: number,
    y?: number,
    width?: number,
    height?: number,
    rotation?: number,
    opacity?: number,
    visible?: boolean,
    locked?: boolean,
    // ... many more properties
})

// Delete nodes
await framer.removeNode(node: CanvasNode)
```

#### Project & User Info
```typescript
// Get project information
const project = await framer.getProject()

// Get current user
const user = await framer.getUser()
```

### CanvasNode Type
```typescript
interface CanvasNode {
    id: string
    name: string
    type: "Frame" | "Text" | "SVG" | "Image" | "Component" | "ComponentInstance" | ...
    x: number
    y: number
    width: number
    height: number
    rotation: number
    opacity: number
    visible: boolean
    locked: boolean
    parent?: CanvasNode
    children?: CanvasNode[]
    // ... many more properties
}
```

## Common Patterns

### React Hook for Selection
```typescript
function useSelection() {
    const [selection, setSelection] = useState<CanvasNode[]>([])
    
    useEffect(() => {
        return framer.subscribeToSelection(setSelection)
    }, [])
    
    return selection
}
```

### Async Operations with Error Handling
```typescript
const handleAction = async () => {
    try {
        await framer.addSVG({ svg: "...", name: "My SVG" })
        framer.notify("Success!", { variant: "success" })
    } catch (error) {
        console.error(error)
        framer.notify("Error occurred", { variant: "error" })
    }
}
```

### Plugin Modes
In `framer.json`:
- `"canvas"` - Plugin appears in canvas mode
- `"preview"` - Plugin appears in preview mode
- Both can be specified as an array

## Styling Best Practices

### Tailwind Usage
- Use Tailwind utility classes for styling
- Framer provides base button styles: `framer-button-primary`, `framer-button-secondary`
- Keep plugin UI compact and focused

### Framer CSS Variables (Dark/Light Mode Support)
Framer provides CSS custom properties that automatically adapt to dark and light modes:

```css
/* Color Variables */
--framer-color-tint              /* Primary brand color */
--framer-color-tint-dimmed       /* Dimmed brand color */
--framer-color-tint-dark         /* Dark brand color */

/* Background Colors */
--framer-color-bg                /* Primary background */
--framer-color-bg-secondary      /* Secondary background */
--framer-color-bg-tertiary       /* Tertiary background */

/* Dividers & Borders */

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [julesvcode/framer-plugin-starter-kit](https://github.com/julesvcode/framer-plugin-starter-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
