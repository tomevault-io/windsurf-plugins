---
trigger: always_on
description: CozyUI is a React-based web frontend for **ComfyUI**, a node-based graphical interface for AI model workflows. The application provides a tabbed interface for various generation workflows: Text-to-Image (T2I), Image-to-Video (I2V), Text-to-Video (T2V), Audio generation, and Upscaling.
---

# CozyUI — Technical Documentation

## Overview

CozyUI is a React-based web frontend for **ComfyUI**, a node-based graphical interface for AI model workflows. The application provides a tabbed interface for various generation workflows: Text-to-Image (T2I), Image-to-Video (I2V), Text-to-Video (T2V), Audio generation, and Upscaling.

**Tech stack:** React 19 + TypeScript, MUI (Material-UI), Redux Toolkit, react-hook-form, Dexie (IndexedDB), WebSocket for ComfyUI communication.

---

## Project Structure

```
public/api/          — ComfyUI workflow JSON files (one per tab)
public/conf/         — Configuration files (config.json, config.local.json)
src/
  api/               — API utilities (mergeType, utils)
  components/
    contexts/        — React contexts (WorkflowTabs, Tab, Filter, ResultOverride)
    controls/        — Reusable UI controls (inputs, buttons, layout)
    controls/mask_editor/ — Mask editor components
    history/         — History panel (IndexedDB, import/export)
    settings/        — Settings UI
    tabs/            — Individual workflow tabs (one per generation type)
  hooks/             — Custom React hooks
  i18n/              — Internationalization (node-polyglot)
  redux/             — Redux slices (config, progress, tab, preview)
```

---

## Architecture

### Entry Point — [`App.tsx`](src/App.tsx)

The app wraps everything in a layered provider hierarchy:

```
ThemeContext
  → I18nContextProvider
    → ConfigLoader (side-effect only — loads config)
    → TagLoader (side-effect only — loads autocomplete tags)
    → ResultOverrideContextProvider
      → WSReceiver (WebSocket listener)
      → VerticalBox
        → WorkflowTabsContextProvider
          → WorkflowTabs
            → [tab children — each <WFTab>]
          → Progress
          → InterruptButton
          → FilterContextProvider
            → HistoryPanel
            → AppSettings
```

### Configuration Loading

[`ConfigLoader.tsx`](src/ConfigLoader.tsx) fetches three data sources in sequence:

1. **`conf/config.json`** — Main configuration defining all tabs, their API files, controls mapping, defaults, and result nodes.
2. **`conf/config.local.json`** — Local overrides (merged into main config).
3. **`/api/object_info`** — Fetched from the ComfyUI server at runtime; contains node class signatures (available options for each input).

Config is stored in Redux under `s.config`. The `loaded` array tracks initialization state.

### Tab System

Each workflow tab is defined by:

1. **React component** in `src/components/tabs/<Name>.tsx` — exports a `<WFTab>` element.
2. **API JSON** in `public/api/<name>.json` — ComfyUI workflow definition.
3. **Config entry** in `public/conf/config.json` under `tabs.<key>`.

Tabs are registered in [`App.tsx`](src/App.tsx) by adding `{NameTab}` inside `<WorkflowTabs>`.

#### [`WFTab`](src/components/WFTab.tsx) Component

```tsx
export type receiverParametersType = {
    name: string;
    weight?: number;
    acceptedTypes: string | string[];  // "images", "gifs", "audio"
};

// WFTab is a marker component — returns null, carries metadata
export const WFTab = ({}: {
    label: string;      // Display name
    value?: number | string;  // config.json key
    content: ReactNode; // The actual content
    group?: string;     // "T2I", "I2V", "T2V", etc.
    receivers?: receiverParametersType[];  // Data receivers from other tabs
}) => null;
```

#### [`WorkflowTabs`](src/components/WorkflowTabs.tsx)

The parent container that:
- Manages the active tab state via Redux (`s.tab`).
- Wraps each tab's `content` in `TabContextProvider` (provides `api`, `tab_name`, `controls`).
- Uses `react-hook-form` (`FormProvider`, `useWatch`) to manage all form state.
- Persists form state to IndexedDB for restoration.
- Provides `ValuesRestore` component that handles form initialization and state recovery.

---

## Configuration (config.json)

Located at [`public/conf/config.json`](public/conf/config.json).

### Top-level Structure

```typescript
type configType = {
    tabs: { [tabName: string]: tabConfigType };
    object_info: { [nodeClass]: { ... } };  // Fetched from ComfyUI API
    llm: { [key: string]: llmConfigType };  // LLM providers for chat
    lora_types: { [type: string]: loraDefaults };
    // ... other settings
};
```

### Tab Config (`tabConfigType`)

```typescript
type tabConfigType = {
    api: string;              // Path to API JSON (e.g., "api/flux.json")
    controls: {               // Maps control names to node fields
        [control: string]: controlType;
    };
    result: { id: string; type: string };  // OR array of such objects
    defaults?: Record<string, any>;         // Default values
    handler_options: {
        lora_params: { /* LoRA loader config */ };
        node_params: { /* Sampler/loader node refs */ };
    };
};
```

### Control Mapping (`controlType`)

```typescript
type controlType = {
    id: string;           // "handle" | "skip" | "NODE_ID"
    field: string;        // Input field name on the node

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SD-inst/cozyui](https://github.com/SD-inst/cozyui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
