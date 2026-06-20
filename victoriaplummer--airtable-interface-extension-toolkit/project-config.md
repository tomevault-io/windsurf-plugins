---
trigger: always_on
description: Complete SDK reference for building Airtable Interface Extensions — models, hooks, components, field types, write patterns, and common pitfalls.
---


# Airtable Custom Interface Extensions — SDK Reference

You are building Airtable Interface Extensions. These are React components that run inside Airtable Interfaces. They use a specific SDK (`@airtable/blocks/interface`) — NOT the older Blocks SDK or the REST API.

## Quick Start Scaffold

```tsx
import {initializeBlock} from '@airtable/blocks/interface/ui';
import React from 'react';

function App() {
    return <div>Hello world</div>;
}

// Entry point — this is NOT ReactDOM.render
initializeBlock({interface: () => <App />});
```

**CLI setup**: `npm install -g @airtable/blocks-cli` → `block init NONE/blkXXX --template=<template-url> my-extension` → `cd my-extension` → `block run`

**Requires**: Node 22+, PAT token with `block:manage` scope configured via `block set-api-key`.

---

## Import Map

Everything comes from two paths:

```tsx
// Models & types
import {FieldType} from '@airtable/blocks/interface/models';

// Hooks, components, utils — everything UI
import {
    useBase, useRecords, useCustomProperties, useGlobalConfig,
    useSynced, useSession, useRunInfo, useColorScheme, useWatchable,
    CellRenderer, expandRecord, initializeBlock,
    colors, colorUtils,
    loadCSSFromString, loadCSSFromURLAsync, loadScriptFromURLAsync,
} from '@airtable/blocks/interface/ui';
```

There is NO Airtable-provided UI component library beyond `<CellRenderer>`. For buttons, inputs, selects, dialogs, etc., use a third-party library like **MUI** (Material UI — used in Airtable's own sliding-bar-chart example) or plain HTML/React elements. See the Styling & External Libraries section for options.

### Old Blocks SDK vs New Interface Extensions SDK

Claude's training data contains extensive examples from the **old** `@airtable/blocks` SDK. Do NOT use those patterns. Key differences:

| Old Blocks SDK (`@airtable/blocks/ui`) | New Interface Extensions (`@airtable/blocks/interface/ui`) |
|---|---|
| `<Button>`, `<Input>`, `<Box>`, `<FormField>`, `<Select>`, `<Dialog>`, `<Tooltip>`, `<Icon>`, etc. | **None of these exist.** Only `<CellRenderer>`. Use plain HTML/React. |
| `import {useBase} from '@airtable/blocks/ui'` | `import {useBase} from '@airtable/blocks/interface/ui'` |
| `initializeBlock(() => <App />)` | `initializeBlock({interface: () => <App />})` |
| `useRecords(queryResult)` with views/sorts/fields | `useRecords(table)` — table-level only, no view access |
| `cursor`, `viewport`, `useViewport` | Not available |

---

## Styling & External Libraries

### npm packages work

The blocks CLI uses webpack. Any npm package compatible with webpack works. Install via `npm install` in your extension directory.

### Recommended libraries

| Need | Recommended | Notes |
|------|-------------|-------|
| **Charts** | `recharts` (React), `d3` + `d3-cloud` | Airtable's official word-cloud example uses D3. Recharts is easier for standard charts. |
| **Date handling** | `date-fns` or `dayjs` | Lighter than moment.js. Date fields return ISO 8601 strings. |
| **Component library** | `@mui/material` (MUI) | Airtable's own sliding-bar-chart example uses MUI v7 + Emotion. Full component set (buttons, inputs, dialogs, etc.). |
| **Headless UI** | `@radix-ui/react-*` | Accessible primitives (dialogs, dropdowns, tooltips) without styling opinions. Lighter than MUI. |
| **Icons** | `@phosphor-icons/react` | Append `Icon` suffix when importing: `import {ArrowRightIcon} from '@phosphor-icons/react'`. |
| **Drag & drop** | `@dnd-kit/core` | Accessible drag-and-drop for kanban boards, sortable lists, etc. |
| **Markdown** | `marked` | Parse markdown content from rich text or long text fields. |
| **3D models** | `@google/model-viewer` | Render 3D models inline. |
| **CSS framework** | Plain CSS or **Tailwind CSS** | Tailwind is officially supported — used in Airtable's own map extension. See Tailwind section below. |

**React 19 note:** If a third-party library doesn't list React 19 as a peer dependency, use `npm install --legacy-peer-deps` to install it.

### CSS approach

Import CSS files directly — the webpack bundler handles them:

```tsx
import './style.css';
```

For external CSS:
```tsx
await loadCSSFromURLAsync('https://cdn.example.com/library.css');
```

For dynamic CSS:
```tsx
loadCSSFromString(`
    .my-card { border: 1px solid #ddd; border-radius: 8px; padding: 16px; }
    @media (prefers-color-scheme: dark) {
        .my-card { border-color: #444; background: #2a2a2a; }
    }
`);
```

### Tailwind CSS — officially supported

Tailwind works with the blocks CLI. Airtable's own [map extension](https://github.com/Airtable/interface-extensions-map) uses this exact setup. The webpack bundler auto-detects PostCSS when the loaders are installed.

**Setup:**
```bash
npm install -D tailwindcss postcss postcss-loader css-loader style-loader autoprefixer @airtable/blocks-webpack-bundler
```

**tailwind.config.js** (at project root):
```js
module.exports = {
    // CRITICAL: must be 'media', not 'class'. Airtable controls dark mode via
    // prefers-color-scheme, not a CSS class. Using 'class' means dark: utilities
    // won't fire unless you manually add a 'dark' class wrapper — and even then

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [victoriaplummer/airtable-interface-extension-toolkit](https://github.com/victoriaplummer/airtable-interface-extension-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
