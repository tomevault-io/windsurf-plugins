---
trigger: always_on
description: Design system basics and living patterns; agents extend this file when UI conventions stabilize. Use with ui-guidelines for all UI work.
---


# Design system (starter)

This file is **intentionally small**. It grows as we lock patterns. **Interaction, motion, a11y, and hard constraints** stay in `.cursor/rules/ui-guidelines.mdc` and `.cursor/rules/01-MUST-DO.mdc`—follow those first.

## Foundations

- **Contrast:** In `apps/dashboard/app/globals.css`, **`muted-foreground`** and **`secondary-foreground`** (dark) are tuned so secondary copy on fills meets **WCAG 2.1 AA** for normal text. **`--border`** stays **subtle** on purpose (light dividers, not high-chroma edges); do not darken it for “contrast” without review—prefer spacing, surface, or focus rings. New semantic colors: verify with a checker (e.g. WebAIM) before shipping.
- **Surfaces:** Prefer theme / Tailwind tokens; one accent per view; no new gradients unless explicitly requested (see ui-guidelines).
- **Radius:** `rounded` only (per project MUST-DO).
- **Spacing:** Default Tailwind scale; avoid arbitrary spacing unless there is a clear, reusable reason.
- **Typography:** Headings `text-balance`, body `text-pretty`, numeric data `tabular-nums` (see ui-guidelines).
- **Primitives:** Use existing app primitives (Base UI / Radix / project components) before inventing new ones (see ui-guidelines).

## Detail page metadata bars

- Use a compact inline `flex` bar at `min-h-10` / `py-2.5` (40px, matching sidebar item height and the 10px spacing grid) for metadata like status, frequency, timestamps.
- Stat items are inline `flex items-center gap-1.5` with `text-xs` labels in `text-muted-foreground` and `font-medium` values in `text-foreground`.
- Status uses a dot (`size-1.5 rounded`) + colored text — not a `Badge` — for compactness.
- Use `flex-wrap` with `gap-x-5 gap-y-1` so the bar wraps cleanly on mobile without breaking the 10px grid.
- Canonical reference: `apps/dashboard/app/(main)/monitors/[id]/page.tsx` → `StatusIndicator` + stats bar.

## Composables v2 — `List` (list pages)

**Prefer** `components/ui/composables/` for new list and chart shells — this is the v2 pattern meant to replace ad-hoc list/chart layouts elsewhere.

All list views use the `List` compound component (`components/ui/composables/list.tsx`). Wrap in `<List className="rounded bg-card">` — no border on root.

- **Row defaults:** `List.Row` defaults to `align="center"` (vertical center). Use `align="start"` when multi-line text should stay top-aligned (monitors, goals, anomalies). `w-full` is built-in — don't repeat it.
- **Cell defaults:** `shrink-0` and `min-w-0` are built-in — don't repeat them. `text-start` is the browser default — don't add it.
- **Inactive/paused state:** `opacity-50` on the Row, not a badge.
- **Cell vertical alignment:** `ListCell` is `flex items-center` by default — content is always vertically centered. **Do not** add `pt-0.5` or other manual padding hacks to align cells. If you need top-alignment, set `align="start"` on the Row and override individual cells with `items-start`.
- **Icon containers:** `size-8 rounded` with semantic `bg-{color}-500/10 text-{color}-600 dark:text-{color}-400`. Use a `TYPE_CONFIG` const map when multiple types exist.
- **Skeletons:** plain `div`s, not `List.Row`. Merge name + secondary into one `flex-1` block.
- **Status badges:** Use `Badge` with `variant="green"` / `"amber"` / `"secondary"` for row-level status (`Active`, `Paused`, `Empty`). Keep it to one badge per row.
- **Description as secondary text:** Show descriptions as a secondary `text-xs text-muted-foreground` line *below* the name in the same cell — not in a separate cell. This avoids wasting a column on optional text.
- **Slug / URL display:** For short identifiers (slugs), show just `/{slug}` in the grow cell — not the full URL. Reserve full URLs for the dropdown menu / external link action.
- **Derived counts from RPC:** When a list page needs child counts (e.g. "3 monitors"), include them in the list RPC response (join + count server-side) rather than making N+1 queries or fetching full relations. Keep the response shape flat (`monitorCount: number`).
- Canonical refs: `monitor-row.tsx`, `status-page-row.tsx`, `funnel-item.tsx`, `goal-item.tsx`, `flags-list.tsx`.

## Composables v2 — `Chart` (chart shells)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [databuddy-analytics/Databuddy](https://github.com/databuddy-analytics/Databuddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
