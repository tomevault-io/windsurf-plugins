---
trigger: always_on
description: When you provide instructions in addition to technical stuff, also include product manager ready instructions.
---

# AI Agent Instructions

When you provide instructions in addition to technical stuff, also include product manager ready instructions.

## Boxento Widget UX Contract

Use this file as the source of truth when creating, updating, or reviewing Boxento widgets.

### Source Of Truth In Code

Read these files before making widget UX decisions:

- `docs/WIDGET_DEVELOPMENT.md`
- `src/components/widgets/common/WidgetShell.tsx`
- `src/components/widgets/common/WidgetSettingsDialog.tsx`
- `src/components/widgets/common/widgetHeaderStyles.ts`
- `src/components/dashboard/DashboardWidgetFrame.tsx`
- `src/components/widgets/index.ts`

Use these widgets as implementation anchors for size-spectrum behavior:

- `src/components/widgets/TodoWidget/index.tsx`
- `src/components/widgets/NotesWidget/index.tsx`
- `src/components/widgets/WeatherWidget/index.tsx`
- `src/components/widgets/QuickLinksWidget/index.tsx`
- `src/components/widgets/RSSWidget/index.tsx`

### Core Principle

Design every widget as a size spectrum, not a single squeezed layout:

1. `1x1` glance state
2. compact widget state
3. standard widget state
4. panel state
5. app state

As the widget gets larger, reveal more workflow, context, and controls.
Do not simply scale the same crowded UI up or down.

### Shell Rules

- Prefer `WidgetShell` for new or heavily refactored widgets.
- Keep the root layout aligned with Boxento's flex model: `widget-container`, `flex`, `h-full`, `flex-col`.
- Keep the content area resilient to resizing with `min-h-0`, `flex-1`, and deliberate overflow behavior.
- Use `contentClassName` on `WidgetShell` to adjust density by size instead of creating a second shell system.
- Keep the header visually integrated with content. Do not add a bottom border or separator line under the header.
- Keep the settings entry point in the header unless the widget is tiny or fully app-sized and the app layout already has a clearer control surface.

### Header Rules

- Hide the outer header in tiny mode.
- Prefer hiding the outer header in app mode when the app surface has its own title, toolbar, tabs, or filters.
- Use compact header spacing only when the widget is short or dense enough that the default header would crowd the content.
- Use a clear, short title that matches the widget's user value.
- Keep header actions minimal. The settings button is standard; add other actions only when they are truly primary.
- In one-column, tiny, or short-row sizes, do not show clipped titles such as `Weat...`. Hide the outer header and expose settings through a size-appropriate overlay/control when the standard header cannot fit.

### Size Rules

#### Tiny: `width === 1 && height === 1`

- Show one glanceable signal, one icon-led cue, or one tap target.
- Do not render dense forms, lists, or multiple competing actions.
- Do not render the standard header.
- Keep text extremely short and centered or intentionally aligned.
- If the only useful action is setup, opening settings from the tiny surface is acceptable.

#### Short Row: `height === 1 && width > 1`

- Use a compact header or compact inline summary.
- Show one-line status, metric, or next action.
- Avoid multi-line content that will overflow vertically.

#### Compact: typically `width <= 2` or `height <= 2`

- Prioritize the primary status or one small workflow.
- Reduce metadata and secondary controls.
- Favor truncation, chips, counters, or compact list items over paragraphs.
- Avoid tables, large forms, and nested panels.

#### Standard: typical default widget sizes

- Show the primary workflow clearly.
- Include the most important controls and enough context to understand the data.
- Keep density moderate and scanning easy.

#### Panel And App: typically `width >= 4 && height >= 4`, especially `width >= 6 && height >= 6`

- Expand into search, filters, tabs, secondary metadata, detail panes, inline editing, or richer previews when the widget's job benefits from them.
- Promote app-specific controls only when the larger surface materially improves the workflow.
- Avoid leaving large surfaces visually empty just because the small widget layout was stretched.

### State Rules

Support these states intentionally:

- loading
- empty
- error
- configured success
- read-only

#### Loading

- Rely on the dashboard `Suspense` fallback for component-load time.
- Use skeletons or subtle in-widget loading states for data refreshes, not full-screen spinners by default.
- Keep loading layout close to final layout to reduce visual jump.

#### Empty

- Explain what is missing in plain language.
- Offer the next step: configure, add an item, connect an account, or choose a source.
- Keep empty states calm and informative instead of decorative.

#### Error

- Show a scoped message tied to the failed task.
- Offer a recovery path when possible: retry, reconfigure, reconnect, or inspect settings.
- Do not dump raw error objects into the widget UI.

#### Read-Only

- Respect `config.readOnly` as a first-class mode.
- Hide settings, editing, creation, reorder, and destructive controls when the widget is read-only.
- Keep the widget fully legible and useful even when interaction is suppressed.

### Settings Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sushaantu/boxento](https://github.com/sushaantu/boxento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
