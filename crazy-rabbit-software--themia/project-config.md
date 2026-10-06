---
trigger: always_on
description: A Windows desktop widget application built with **Tauri 2** (Rust backend) and **SolidJS** (frontend). Widgets render as transparent, always-on-desktop windows using `WS_EX_NOACTIVATE` for a non-intrusive experience.
---

# Desktop Widget App

## Project Overview
A Windows desktop widget application built with **Tauri 2** (Rust backend) and **SolidJS** (frontend). Widgets render as transparent, always-on-desktop windows using `WS_EX_NOACTIVATE` for a non-intrusive experience.

## Tech Stack
- **Frontend**: SolidJS 1.8, TypeScript, Vite 8
- **Backend**: Rust (Tauri 2)
- **Icons**: lucide-solid (use this for all icons, never emojis in code)
- **Styling**: Plain CSS files (no CSS-in-JS, no Tailwind)

## File Structure
```
src/
  main.tsx              # Entry point
  App.tsx               # Root component (view management, design panel, widget picker)
  WidgetGrid.tsx        # Grid layout with drag/resize
  ContextMenu.tsx       # Global context menu renderer
  layout.ts             # Widget layout state (positions, sizes)
  design.ts             # Design system state (colors, gaps, corners)
  views.ts              # Multi-view state management
  ctxMenuStore.ts       # Context menu store (showCtxMenu, closeCtxMenu)
  fileClipboard.ts      # Copy/cut/paste state for file operations
  widgets/
    registry.tsx        # Widget registry: all widget types registered here

widgets/
  email/                # Email widget (Microsoft Graph + IMAP)
    Email.tsx
    EmailConfig.tsx
    Email.css
    EmailConfig.css
    src/lib.rs          # Rust backend (accounts, messages, auth)
    Cargo.toml
  folder/               # File explorer widget
    FolderView.tsx
    FolderViewConfig.tsx
    FolderView.css
    FolderViewConfig.css
  contacts/             # Contacts widget (Microsoft Graph)
    Contacts.tsx
    ContactsConfig.tsx
    Contacts.css

src-tauri/
  src/lib.rs            # Tauri command registration
```

## Design System

### CSS Variables (dark theme)
All widgets use these CSS variables; never hardcode colors:
```css
--foreground:        rgba(255, 255, 255, 0.886)   /* Primary text */
--muted-foreground:  rgba(255, 255, 255, 0.544)   /* Secondary text */
--subtle-foreground: rgba(255, 255, 255, 0.36)    /* Tertiary/preview text */
--card:              rgba(30, 30, 30, 0.90)        /* Widget background */
--card-hover:        rgba(255, 255, 255, 0.06)     /* Hover state */
--border:            rgba(255, 255, 255, 0.08)     /* Borders */
--primary:           #60cdff                       /* Accent (cyan) */
--destructive:       #ef4444                       /* Error/danger (red) */
--radius:            8px                           /* Default border radius */
--item-gap:          0px                           /* Gap between list items */
```

### Styling Conventions
- Each component has a companion `.css` file (e.g., `Email.tsx` + `Email.css`)
- Use a unique root class per component to scope styles (e.g., `.email-widget`, `.email-config-body`)
- Glass effect: `backdrop-filter: blur(60px) saturate(180%)`
- Interactive elements: `transition: background 0.12s` with `rgba(255,255,255,0.06)` hover
- Buttons: transparent background, 1px solid border with low-opacity white, `font-family: inherit`
- Scrollbars: config/settings surfaces (`.cfg`, settings modal, config window) use an always-visible styled thin scrollbar (`overflow-y: scroll` + `scrollbar-width: thin` + styled `::-webkit-scrollbar-thumb`). Widget display bodies AND dropdown/picker panels (`.dropdown-panel`, `.icon-picker-panel`) hide scrollbars (`scrollbar-width: none` + `::-webkit-scrollbar { display: none }`)
- Scroll fade: gradient overlay at bottom of scrollable widget lists

### Typography
- Font sizes: 10px (labels/badges), 11px (preview), 12px (body), 13px (controls/inputs)
- Font weights: 400 (normal), 500 (medium), 600 (semibold), 700 (bold/badges)
- Use `font-variant-numeric: tabular-nums` for dates/numbers

## Widget Architecture

### Registry Pattern (`src/widgets/registry.tsx`)
Every widget type is registered with:
```ts
{
  component: Component,           // Main display component
  configComponent: ConfigComponent, // Settings panel component
  label: string,                   // Display name
  icon: string,                    // Emoji icon for picker
  category: string,                // Grouping category
  defaultSpan: { cols, rows },     // Default grid size
  defaultConfig: {},               // Initial config values
  contextActions: (config, setConfig) => CtxMenuItem[],  // Right-click menu
}
```

### Widget Props Pattern
Display components receive config values as flat props:
```tsx
export default function MyWidget(props: MyWidgetProps) { ... }
```

Config components receive:
```tsx
interface ConfigProps {
  config: Record<string, any>;
  onChange: (config: Record<string, any>) => void;
  globals?: Record<string, any>;
}
```

### Context Menu Pattern
Two levels of context menus:
1. **Widget-level**: Defined in `registry.tsx` via `contextActions`. Triggered by right-clicking the widget chrome.
2. **Item-level**: Defined inside the component. Triggered by right-clicking individual items (emails, files, contacts).

```tsx
import { showCtxMenu } from '../../src/ctxMenuStore.ts';

function handleItemRightClick(e: MouseEvent, item: Item) {
  showCtxMenu(e, [

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crazy-rabbit-software/themia](https://github.com/crazy-rabbit-software/themia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
