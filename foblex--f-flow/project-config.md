---
trigger: always_on
description: Use this file when adding, rewriting, or reviewing examples inside `libs/f-examples`.
---

# AGENT.md

## Purpose

Use this file when adding, rewriting, or reviewing examples inside `libs/f-examples`.
This guide is based on the examples that already follow the newer repository style and on the way examples are wired into the docs portal.

## Source of truth

Treat these examples as the primary reference for new work:

- `libs/f-examples/advanced/copy-paste/example.ts`
- `libs/f-examples/advanced/flow-state/example.ts`
- `libs/f-examples/connections/connection-content/example.ts`
- `libs/f-examples/connectors/connection-rules/example.ts`
- `libs/f-examples/extensions/auto-pan/example.ts`
- `libs/f-examples/nodes/custom-nodes/example.ts`
- `libs/f-examples/nodes/grouping/example.ts`

Treat these as additional references for more stateful or event-heavy examples:

- `libs/f-examples/connectors/node-as-connector/example.ts`
- `libs/f-examples/extensions/selection-area/example.ts`
- `libs/f-examples/advanced/custom-event-triggers/example.ts`

## Canonical example shape

- Place each example in `libs/f-examples/<category>/<slug>/`.
- Use the existing top-level categories:
  - `nodes`
  - `connectors`
  - `connections`
  - `extensions`
  - `plugins`
  - `advanced`
- Keep the normal example folder self-contained and flat.
- The common folder shape is exactly three root files:
  - `example.ts`
  - `example.html`
  - `example.scss`
- Do not add `index.ts` to example folders. Current examples do not use local barrels.
- Add extra local files only when the example is too complex for a single component file. Keep those helpers colocated under the same example folder.

## New-style naming

- Use the folder slug as the source of truth for naming.
- Always use the fixed entry file names:
  - `example.ts`
  - `example.html`
  - `example.scss`
- Do not create new examples as `<slug>.ts`, `*.component.ts`, `*.directive.ts`, or `*.service.ts`.
- Keep the selector equal to the folder slug:
  - `selector: 'auto-pan'`
  - `selector: 'copy-paste'`
- Name the entry component class `Example` in every example folder.
- Keep the template and style names on the fixed `example.*` convention.

## Component structure

- Make every example a standalone Angular component.
- Use `ChangeDetectionStrategy.OnPush`.
- Import `FFlowModule` in the example component. Every current example does this.
- Add extra imports only for the specific demo need:
  - Angular Material controls
  - `@foblex/m-render` control helpers such as `FToolbarComponent`, `FSelectComponent`, `FInputComponent`, `FExternalPaletteComponent`
  - extra Foblex declarables when the example explicitly demonstrates them
- Keep example-only interfaces and lightweight helper types in the same `.ts` file unless the example becomes genuinely multi-part.

## State and DI

- Prefer function-based Angular APIs in new examples:
  - `viewChild(...)`
  - `signal(...)`
  - `model(...)`
  - `computed(...)`
  - `inject(...)`
- Prefer `private readonly _canvas = viewChild(...)` or `viewChild.required(...)` for canvas/flow access.
- Prefer `protected readonly` state for values that are read in the template.
- Prefer `private readonly _...` fields for internal references and helpers.
- Avoid public mutable template-facing fields in new examples.
- Use simple local state that explains the feature directly. Do not introduce app-like store layers unless the example is explicitly about state/history patterns, as in `flow-state`.

## Methods

- In new examples, prefer action names without the `on` prefix:
  - `loaded`
  - `createConnection`
  - `reassignConnection`
  - `selectionChanged`
  - `canvasChanged`
  - `deleteConnections`
- Keep private helpers underscore-prefixed and verb-based:
  - `_hasConnection`
  - `_copyInternal`
  - `_applyChanges`
  - `_removeConnection`
- Keep event handlers short and feature-oriented. Push complex steps into small private helpers.
- Use public methods only for Angular lifecycle hooks such as `ngOnInit` or `ngOnDestroy`.

## Template structure

- Start the example with `<f-flow>` and keep `<f-canvas>` inside it.
- Bind example outputs at the flow/canvas level and forward them to local action methods.
- Use Angular built-in control flow. Current examples use `@for`, `@if`, and tracking expressions.
- Do not introduce `*ngIf`, `*ngFor`, or `*ngSwitch` in new example templates.
- When rendering node or connection collections, use `@for (...; track ...)`.
- If the example has controls, place them outside the flow in `<f-toolbar>`.
- If the example has an external drag source or other surrounding UI, keep that UI outside the flow and colocated in the same template.

## Event API usage

- Prefer current event payload names in new examples when the event already exposes them:
  - `sourceId`
  - `targetId`
  - `dropPosition`
  - `previousSourceId`
  - `nextSourceId`
  - `previousTargetId`
  - `nextTargetId`
  - `nodeIds`
  - `connectionIds`
- Do not introduce new examples that depend on deprecated aliases like:
  - `fOutputId`
  - `fInputId`
  - `fDropPosition`
  - `oldSourceId`
  - `newTargetId`
  - `fNodeIds`
  - `fConnectionIds`
- Legacy examples still use those aliases. Treat that as migration debt, not as the target style for new examples.

## Example behavior and UX


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Foblex/f-flow](https://github.com/Foblex/f-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
