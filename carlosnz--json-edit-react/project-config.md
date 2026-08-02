---
trigger: always_on
description: Guidance for Claude Code when working in this sub-package.
---

# CLAUDE.md — @json-edit-react/components

Guidance for Claude Code when working in this sub-package.

## What this is

A published package of pre-built custom node components for [`json-edit-react`](https://github.com/CarlosNZ/json-edit-react). Each component is a React component + a `CustomNodeDefinition` that the consumer drops into the editor's `customNodeDefinitions` prop.

## Public API

Re-exported from [src/index.ts](src/index.ts). Each component lives in its own folder under `src/` with the structure `{component.tsx, definition.ts, index.ts}` (plus `style.css` for some).

**Exception — the swappable widgets ship under their own subpath**, `@json-edit-react/components/widgets` (`src/widgets/`), and are deliberately NOT re-exported from the root. They are a different *kind* of thing from the node-definition components — they have no `definition.ts`, they satisfy a props contract and get *passed in* to be rendered, rather than registering as a node type. Two flavours live here:

- **Editor-slot widgets** (`ReactSelect`, `CodeEditor`) satisfy a core contract (`SelectProps`, `TextEditorProps`) and get passed to JsonEditor's top-level `Select` / `TextEditor` props to replace a built-in UI control.
- **Node-component widgets** (`ReactDatePicker`) satisfy a contract defined *in this package* (`DatePickerWidgetProps`, in `_common/`, re-exported from the root) and get passed to a node component via `componentProps` (e.g. `datePickerDefinition({ componentProps: { DatePicker: ReactDatePicker } })`). This keeps a heavy picker dep opt-in and lets a consumer drop in their own picker — the `DatePicker` node falls back to a `StringEdit` + warning when none is supplied. The contract is value-agnostic (`Date` in / `Date` out), so a single widget can serve any future date-shaped node type.

Splitting them out keeps the root barrel uniformly node-definition components. This is purely a **conceptual** grouping — NOT a tree-shaking measure (the widgets' heavy libs are already `React.lazy`-loaded and the wrapper code is tiny); don't conflate it with the bundle-bloat escape hatch below. The wiring lives in the same three places as a future sub-path group: `package.json` `exports` (+ a `typesVersions` fallback for classic `moduleResolution: node`), a second `jsBundle`/`dtsBundle` pair in [rollup.config.mjs](rollup.config.mjs), and the demo's Vite alias (`componentsWidgetsSrcMap`).

## Conventions

### Third-party deps strategy: Option B+

- All third-party libs (`react-datepicker`, `react-markdown`, `react-colorful`, `colord`, `use-debounce`) are **regular `dependencies`**, not optional peers.
  - Consumers install the package once and get everything that comes with it — no per-component "you must also install X" friction.
- The package ships ESM + CJS with `"sideEffects": false`, and the build stamps `/*#__PURE__*/` annotations (see Build) so modern bundlers (Webpack 4+, Vite, Rollup, esbuild, Parcel 2+) tree-shake unused components out of the consumer's bundle. `sideEffects: false` alone is **not** enough here: the whole package bundles into one ESM file, so a flag that governs dropping whole *modules* can't drop unused components *within* that one file — only the per-call purity annotations can (issue #388).
- **Heavy components use `React.lazy(() => import('...'))`** for their third-party dep — `DatePicker`, `ColorPicker`, `Markdown`. This means even when a consumer imports the component, the heavy library only loads at the moment the component is first rendered. The component's chunk lives in `node_modules` (install cost) but contributes nothing to the initial JS bundle (runtime cost).
- **`react` and `json-edit-react` are peer dependencies.** Consumers supply them.

### Future escape hatch: sub-path exports

The root barrel now tree-shakes per component via the `/*#__PURE__*/` annotations (issue #388), so a selective `import { hyperlinkDefinition } from '@json-edit-react/components'` no longer pulls the other components or their heavy deps. Sub-path exports remain a possible future escape hatch — e.g. for legacy CJS consumers whose bundler can't tree-shake ESM, or to drop the few exotic-type definitions that still ride along until their eager `defaultValue` calls are de-eagered (see "Adding a new component"). The planned migration if it's ever needed:

1. Add per-component sub-path exports in [package.json](package.json) `exports`:
   ```json
   "./hyperlink": { "import": "./build/hyperlink.esm.js", "require": "./build/hyperlink.cjs.js" },
   "./datepicker": { ... },
   ...
   ```
2. Update [rollup.config.mjs](rollup.config.mjs) to a multi-entry build with one entry per component.
3. **Non-breaking** — the root `.` entry stays, so existing `import { Hyperlink } from '@json-edit-react/components'` keeps working.

This isn't worth doing pre-emptively. Wait for a real consumer report. See `dev-docs/package-management-guide.md` for the bundle-size test scaffolding plan.

### What NOT to do

- Don't add `peerDependenciesMeta` with `optional: true` for the heavy libs — we deliberately chose regular deps for ergonomics.
- Don't bundle the third-party deps into the output — they stay external. The `external` list in rollup.config.mjs is the source of truth for this.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CarlosNZ/json-edit-react](https://github.com/CarlosNZ/json-edit-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
