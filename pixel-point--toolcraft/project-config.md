---
trigger: always_on
description: <!-- toolcraft-performance-lifecycle: first-delivery=functional-complete; later-edits=focused-only; complaint=one-authority-targeted-performance-iteration; full-audit=explicit-only -->
---

# Toolcraft App Template Assembly Guide

<!-- toolcraft-performance-lifecycle: first-delivery=functional-complete; later-edits=focused-only; complaint=one-authority-targeted-performance-iteration; full-audit=explicit-only -->

This is a standalone Toolcraft template app generated from the base starter.

## Required Preflight

Treat this `AGENTS.md` as the active project contract. Before planning or editing app code, runtime code, controls, canvas, panels, renderer, timeline, layers, export, or tests:

1. Read `docs/toolcraft/workflow.md` in full.
2. Select every task route that matches the requested surface.
3. Read the selected routes' Plan phase as contract context before implementation; it does not require a plan document. Use the proportional planning policy below to decide whether a written plan is warranted.
4. Read their Implementation phase immediately before editing code.
5. Read their Verification phase before writing or running proof.

Process matching routes sequentially within the current phase and skip repeated documents already read in that phase. Open exactly one listed document per terminal or tool read, including documents from the same route and phase. Do not concatenate multiple documents, routes, or phases into one large output, and never continue from truncated output. Choose and record one verification tier before first product delivery; for later edits record only the focused checks selected for the changed behavior. Do not edit implementation files until the Plan and Implementation preflight is complete.

## Quick Entry Contract

Runtime owns the evaluated finite/Infinity Background below runtime model/image layers; product output stays a transparent foreground and never recreates a bounded preview background. Compound `collectionActions` fields opt into keyframes individually with `keyframeable: true`; selectable collections declare one nullable `selectionTarget`, and panel/canvas code dispatches structured collection commands instead of encoding nested timeline addresses or mutating arrays.

1. Build through `defineToolcraft({ base, modules })` and `composeToolcraftApp(appSchema, ports)`; the signed route alone renders `ToolcraftApp`. Preserve the generated `app-defaults.json` import and `defaults: appDefaults` input; local Setup saves source defaults through the host-owned path in `docs/toolcraft/core/setup-export.md#save-app-defaults`.
2. Keep app state in Toolcraft runtime schema and commands.
3. Keep product output in `scene.canvasContent`; never render app UI there. The signed host owns `ToolcraftApp`, bootstrap, routes, global runtime styles, and one canonical product scene across finite and infinite modes; product code supplies ports only through `composeToolcraftApp`. `scene.sceneBoundsProvider` declares the product frame in both modes, finite mode shows the centered artboard boundary and clipping, and Infinity removes only their on-screen presentation and the size controls while preserving view, world frames, component/renderer identity, and live backing. Product export receives the same provider rect as live output; only finite mode may fall back to its artboard rect when no provider exists. Image source pixels and image scene geometry are distinct. Both modes export the saved centered finite artboard with the same crop, dimensions, content and background for identical scene/export settings; Infinity never expands export to the visible contributor union. Custom raster/WebGL output consumes `useToolcraftProductSceneFrame`. A preview-only environment that must fill the complete Infinity viewport uses `scene.infiniteCanvasContent`; it remains pointer-transparent and outside world transforms, scene bounds, and export. Model upload preserves supported authored appearance from standalone files, folders, and bounded ZIP packages; omit `modelPresentation` for runtime ownership and use checked consumers only for custom presentation. `scene.renderDefaultCanvasMedia: false` never hides runtime model layers. If upload/import is part of the source-material flow, do not invent canvas placeholder artwork, CTA copy, helper text, fake sample output, or preset source designs before real content exists.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pixel-point/toolcraft](https://github.com/pixel-point/toolcraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
