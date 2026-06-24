---
trigger: always_on
description: This document provides context for AI models (Gemini, GPT, etc.) to understand the codebase structure, data models, and business logic of the **LVGL Studio AI** application.
---

# Project Context: LVGL Studio AI

This document provides context for AI models (Gemini, GPT, etc.) to understand the codebase structure, data models, and business logic of the **LVGL Studio AI** application.

## 1. Overview

LVGL Studio AI is a React 19 web application that allows users to visually design embedded GUIs for the LVGL (Light and Versatile Graphics Library) framework. It generates C or MicroPython code based on the visual design.

## 2. Tech Stack

- **Frontend**: React 19, TypeScript, TailwindCSS.
- **Icons**: Lucide React.
- **AI SDK**: `@google/genai` (Gemini).
- **Build System**: ES Modules via `importmap` (No bundler required for dev).
- **Persistence**: LocalStorage for auto-save, JSON file Export/Import for projects.

## 3. Data Model (`types.ts`)

The application state is hierarchical:

```typescript
interface Screen {
  id: string;
  name: string;
  backgroundColor: string;
  layers: Layer[]; // Z-index groups
  widgets: Widget[]; // Flat list of widgets for this screen
}

interface Widget {
  id: string;
  type: WidgetType; // e.g., 'lv_btn', 'lv_label'
  events: WidgetEvent[];
  style: WidgetStyle; // CSS-like props mapped to LVGL styles (includes shadowSpread)
  flags?: WidgetFlags; // LVGL specific flags (hidden, checkable, scrollable, etc.)

  // Image Specifics
  src?: string; // Filename reference (e.g. "icon.png")
  imageData?: string; // Base64 string for Browser Preview ONLY (stripped before sending to AI)

  // List Options (Roller/Dropdown)
  options?: string; // Newline separated string

  // ... coordinates (x, y, width, height)
}

interface ProjectFile {
  version: string;
  screens: Screen[];
  settings: CanvasSettings;
  // ... styles, etc
}
```

## 4. Key Components

### `App.tsx`

- Holds the root state: `screens[]`, `activeScreenId`, `canvasSettings`.
- **State Management**: Uses custom `useHistory` hook to manage `past`, `present` (state), and `future` stacks.
- **Persistence**:
  - Lazily initializes state from `localStorage` on mount.
  - Updates `localStorage` on every state change.
  - Handles JSON file Import/Export logic (`FileReader` / `Blob`).
- Manages global actions (Add Widget, Generate Code, Apply Theme) via `updateProject` wrapper for History tracking.

### `hooks/useHistory.ts`

- Provides generic `undo`, `redo`, `jumpTo`, and `set` capabilities.
- Stores history as `{ state: T, name: string, timestamp: number }` items.

### `components/Canvas.tsx`

- Renders the "Active Screen".
- Handles Drag-and-Drop (creation), Drag-to-Move, and Resizing.
- **Smart Alignment**: Implements a magnetic snap system (5px threshold) that aligns moving widgets to the edges and centers of other widgets. Renders magenta guidelines (`#ec4899`) when snapped.
- **Zooming**: Supports zooming via `Alt + Scroll` (20-300%).
- **Preview Mode**: When `isPreview` is enabled, editing (drag/resize) is disabled. Click events (`onClick`) are captured and bubbled up to `App.tsx`'s `handleWidgetEvent` to trigger actions like `NAVIGATE` without code generation.
- Renders LVGL-like HTML approximations of widgets.
- **Rendering Architecture**: Implements a split-rendering strategy:
  - **Wrapper**: Handles absolute positioning, interactions (click/drag), selection rings, and resize handles. Always has `overflow: visible` to prevent handle clipping.
  - **Visual Box**: Inner container that applies the actual widget styles (background, border, shadow, opacity) and controls content overflow (visible/hidden/scroll).
- **Slider Orientation**: Automatically renders as **Vertical** if height > width.
- **Image Rendering**: Renders Base64 `imageData` if present, otherwise shows a placeholder.

### `components/HelpDialog.tsx`

- Renders a modal overlay with:
  - **Keyboard Shortcuts**: e.g., Undo (Ctrl+Z), Zoom (Alt+Scroll).
  - **Documentation**: Quick tips on widgets and workflow.
  - **Links**: External links to full docs.

### `components/PropertiesPanel.tsx`

- **Selection Mode**: Edits properties of selected widget(s).
- **Global Mode**: When nothing is selected, edits Active Screen settings, Global Theme, and **Layer Management** (Reorder, Lock, Hide).
- **Image Upload**: Handles converting `File` inputs to Base64 strings for preview.

### `components/WidgetPalette.tsx`

- Lists available widgets.
- **AI Widget Generator**: Contains an input field to accept natural language prompts. Calls `generateSingleWidget` service to create widget JSON on the fly.

### `services/aiService.ts`

- **Prompt Engineering**: serialized the `screens` array into a simplified JSON format.
- **Optimization**: Specifically filters out `imageData` (Base64) from the JSON payload to prevent token limit exhaustion, sending only the `src` filename to the LLM.
- **Context Injection**: Instructs the LLM to generate:
  - `ui_init()` function.
  - Event callbacks (`ui_event_Button1`).
  - Image references using `src` filename (e.g., `lv_img_set_src(obj, "S:path/" + src)`).
  - Flags: `lv_obj_add_flag` for properties like `LV_OBJ_FLAG_OVERFLOW_VISIBLE`, `LV_OBJ_FLAG_CHECKABLE`, etc.
  - Styles: Extended support including `shadow_spread`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dazeb/LVGL-AI-Studio](https://github.com/dazeb/LVGL-AI-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
