---
trigger: always_on
description: Read before editing this app. The installed build-dashboard or build-report skill owns analytical composition and copy; this guide owns local file boundaries, styling, protected-runtime changes, and builds. It stays with the copied app. Resolve the component reference through the build workflow below and read only the topics needed for the task.
---

# Data App Authoring Guide

Read before editing this app. The installed build-dashboard or build-report skill owns analytical composition and copy; this guide owns local file boundaries, styling, protected-runtime changes, and builds. It stays with the copied app. Resolve the component reference through the build workflow below and read only the topics needed for the task.

## Editable content

- Put app-specific React, scoped CSS, helpers, and approved assets in `src/content/`. Dashboard and report entry points are `dashboard/DashboardContent.jsx` and `report/ReportContent.jsx` beneath that directory; shared helpers and assets may use `shared/` and `assets/`.
- Update reviewed rows and exact provenance in `src/data.json`. Data corrections are artifact-local unless an external write is explicitly authorized; never misrepresent corrected values as an unchanged external result.
- Theme tokens live in `src/theme.css`. Theme-only edits require no additional confirmation; do not restyle protected chrome through content CSS.
- Import shared charting, components, controls, formatting, and data access only through `src/data-app-public.jsx`. Use the public evidence wrappers and source actions for custom visuals as well.
- The shell owns the only `main` landmark. Authored content uses `article`, `section`, or `div`.

`docs/components/` is a plugin-owned reference snapshot for the copied runtime, outside editable app content. Ordinary app revisions use the component API without modifying these reference files.

## Preserve behavior and identity

Keep the real top bar, theme switching, dashboard refresh, menus, chart editing, source inspection, inline editing, autosave, print/export, and publication/access behavior. Keep the existing project, app ID, published destination, and user presentation during revisions.

Set `surface` in `src/data.json` to `dashboard` or `report`. Each new app needs a fresh stable top-level `id`. Keep it unchanged during data, title, or layout edits. To rename an older app without an ID, assign a new safe ID and record the exact old title as `legacyPresentationTitle`; preserve both in reproducers. Do not scan browser storage, reuse another app's ID, or invent storage migrations. Ordinary title editing uses the shared editor.

Each source-backed component needs a stable `id` and existing reviewed `queryId`. Keep reviewed values, rows, controls, statuses, axes, and computed outputs read-only to inline text editing; mark custom source collections `data-reviewed-rows`. Authored captions/copy may use stable editable IDs; report prose uses `RichNarrative` for the shared formatting toolbar. Tooltip/source metadata is not inline-editable.

Retain the intended movable blocks and owner-only Edit behavior. Use `sortable-layout.md` in the resolved component reference for canvas, freeform, composite, or stack layouts. Keep page controls outside sortable regions. Increment `authoredRevision` only for an explicitly requested rearrangement; routine data, copy, or styling updates preserve user placement. Layout persistence must never mutate reviewed rows or provenance.

## Geometry and themes

Use Classic for unstyled new apps; preserve existing or requested themes.

Standard dashboards have one shell-owned 1440px usable column with 32px desktop / 16px mobile gutters; do not add nested page gutters. Preserve custom full-width/viewport layouts. Full-bleed sections use `--data-app-layout-intent: full-bleed`, retaining aligned inner content. For an explicit custom width/gutter, set `--data-app-layout-intent: user-requested` and `--data-app-content-width` on the authored page. Report defaults and its content-only width option are in the report API.

The protected top bar and tabs remain full-viewport regardless of content width. For app-wide restyling, update the existing `:root` palette in `src/theme.css`, including surfaces, controls, text, borders, and chart colors; never scope the whole palette to a content page. Set `--data-app-chrome-background` and `--data-app-chrome-text` to the desired theme tokens without changing chrome layout or functionality.

Keep metric values and marks contained, controls wrapping, and wide tables scrolling internally. Right-align quantitative table values, left-align text/miniature visuals, and omit redundant axis titles. Maps require projected geography and actual reviewed coordinates; reuse `src/content/dashboard/regional-world-map.js` where suitable, not schematic bubbles presented as geography.

## Protected infrastructure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anonymous-report-421/GPT-as-Policy](https://github.com/anonymous-report-421/GPT-as-Policy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
