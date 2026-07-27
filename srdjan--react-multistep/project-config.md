---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **react-multistep**, a published npm package (v8.x) providing a headless
React component for multi-step forms. The component owns step state, validation
gating, and navigation; the consumer renders all UI (step indicators, prev/next
buttons, panels). It is written in TypeScript and ships as an ESM-only build
targeting React 19.2+.

## Common Development Commands

```bash
npm install        # Install dependencies
npm run build      # esbuild ESM bundle + tsc declarations + copy CSS -> ./dist
npm run typecheck  # tsc over src (NodeNext) and src+test (Bundler)
npm run lint       # eslint over src and test
npm test           # homegrown runner (test/run.mjs): jsdom + react-dom
npm run format     # prettier write over src and test
npm run prepack    # lint + typecheck + test + build (runs before publish)
```

There is a single `dist/` output. There is no separate `build/` tree and no
CommonJS bundle.

### Build pipeline (`npm run build`)

One script does three things:

1. **esbuild** bundles `src/index.ts` to `dist/index.js` as ESM
   (`--format=esm`), with a linked sourcemap, automatic JSX
   (`--jsx=automatic`), and `react` / `react/jsx-runtime` kept external (the peer
   dependency is never bundled).
2. **tsc** emits declarations only (`--emitDeclarationOnly --outDir dist`):
   `dist/index.d.ts` plus a declaration map.
3. A small Node one-liner copies `multistep.css`, `tokens.css`, and `chrome.css`
   from `src/styles/` into `dist/`.

### Working with the Example App

```bash
cd examples/client-side
npm install
npm run dev        # esbuild dev server at http://localhost:8000
```

The single canonical example (`examples/client-side`) is TypeScript and builds
directly from `src`.

## Architecture

### Headless component

The package's default export is `MultiStep` (`src/MultiStep.tsx`). It renders no
chrome. Its job is to:

- Parse `children` into an array of step elements (`React.Children.toArray` +
  `isValidElement`), throwing if there are zero valid children.
- Hold all wizard state in a `useReducer` (`src/MultiStep.tsx`): the active step,
  a per-step `StepValidity[]`, a per-step `visited` boolean array, and the total
  step count. Actions: `SYNC_STEPS`, `SET_ACTIVE`, `SET_STEP_VALIDITY`.
- Support both controlled (`activeStep` + `onStepChange`) and uncontrolled
  (`defaultStep`) operation, with clamping to the valid range. Prop-into-state
  reconciliation (step-count changes, controlled-value sync) happens during
  render via a `reconcile()` helper and stored previous-value refs, not in
  effects, so there are no `useEffect` calls for it and no StrictMode double-fire.
- Derive a `steps: Step[]` metadata array and expose the full `MultiStepApi`
  through context.
- Render steps according to `mode` (see Render mode below), wrapping each rendered
  step in `StepIndexProvider` so `useReportValidity` can resolve its index.

### The validity contract: `useReportValidity`

There is **no injected prop**. A step reports its validity by calling the
`useReportValidity()` hook (`src/MultiStepContext.tsx`) from inside its own
subtree:

```tsx
const report = useReportValidity();
useEffect(() => {
  report({ status: "valid" }); // or { status: "invalid", message?, errors? } / { status: "pending" }
}, [report /* , ...deps */]);
```

The returned callback is stable (memoized on the report channel + step index) and
dispatches `SET_STEP_VALIDITY` into the reducer. `useReportValidity` reads
`StepIndexContext` and `ReportValidityContext`; if either is null it throws the
exact string `useReportValidity must be used within a MultiStep step`.

`StepValidity` (`src/interfaces.ts`) is a discriminated union:
`{ status: "valid" }` | `{ status: "invalid"; message?; errors? }` |
`{ status: "pending" }`. Every step's initial validity is `{ status: "pending" }`,
which is not valid, so the forward gate is blocked until a step reports `valid`.

### Forward gate

`goToStep(target)` (`src/MultiStep.tsx`) allows backward navigation freely and
ignores out-of-range targets. For `target > active`, it requires every step in
`[active, target)` to have status `valid`; if any is not, it calls
`onValidationError(firstInvalidIndex)` and aborts. `next()` and `previous()` go
through `goToStep`. `complete()` finishes the wizard: when the active step is the
last step and `valid` (`canComplete`), it fires `onComplete?()`; otherwise it
calls `onValidationError(activeStep)`. All navigation callbacks (including
`complete`) read every dynamic value from a ref (`navRef`) so they stay
referentially stable across renders.

### Step-change guard (`beforeStepChange`)

An optional `beforeStepChange?: (event: StepChangeEvent) => boolean | void |
Promise<boolean | void>` prop runs after the forward validity gate passes but
before a step change commits (`StepChangeEvent` = `{ from, to, direction }`,
`direction` being `"next" | "previous" | "jump"`). Returning `false` (or a
rejected/throwing guard, which is caught and swallowed) vetoes the change;
anything else proceeds. The guard is awaited inside `goToStep`; while an async

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [srdjan/react-multistep](https://github.com/srdjan/react-multistep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
