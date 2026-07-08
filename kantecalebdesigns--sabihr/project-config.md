---
trigger: always_on
description: This file documents the visual language we've converged on for the SabiHR app. Apply these patterns whenever adding or modifying UI. Auto-loaded by Claude Code every session.
---

# SabiHR — Design System Rules

This file documents the visual language we've converged on for the SabiHR app. Apply these patterns whenever adding or modifying UI. Auto-loaded by Claude Code every session.

## Background

- **All page-level backgrounds**: `bg-[#f7fbff]` (Tailwind `blue-50` shade — soft light blue).
- Applied in: `src/components/layout/app-layout.tsx`, `src/components/layout/employee-layout.tsx`, and any standalone page that sets its own `min-h-screen bg-*`.
- Internal element tints (table hover, nested cards, chip backgrounds): use `bg-slate-50` or `bg-slate-100`, not a second blue shade — reserve blue for brand/primary use.

## Cards

Standard card shell for any content block (tables, KPIs, lists, forms):

```
rounded-2xl border border-slate-200/70 bg-white shadow-[0_1px_2px_rgba(15,23,42,0.04),0_1px_3px_rgba(15,23,42,0.05)]
```

- **Radius**: always `rounded-2xl` (not `rounded-xl` or `rounded-lg`) for top-level cards.
- **Border**: `border border-slate-200/70` (soft, low-contrast — the shadow does the lifting).
- **Shadow**: the long `shadow-[...]` literal above — promote to a utility if we add a Tailwind plugin.
- **Internal dividers** (between sub-sections or table rows): `border-slate-100` (lighter than the outer border).

Never use `border-[#efefef]` for new work — legacy pages still have it; replace on touch.

## Tables

Consistent with the **Employees** page table, which is the reference:

- **Outer**: card shell (see above) with `overflow-hidden`.
- **Toolbar row** (search + count + sort, above the table): `flex items-center gap-3 justify-between px-5 py-4 border-b border-slate-200/70`.
- **Thead row**: just `border-b border-slate-200/70`. No `bg-[#f8fafc]` / no gray header band.
- **Th cells**: `text-left font-medium text-[11px] uppercase tracking-wider text-slate-500 py-3 pr-5`. First column gets `pl-5`.
- **Tbody rows**: `border-b border-slate-100 last:border-0 hover:bg-slate-50/60 transition-colors`. Add `cursor-pointer` when rows navigate.
- **Td cells**: `py-4 pr-5`, first column `pl-5`. Use `py-4` for breathing room.
- **Checkbox column**: `w-12 pl-5 py-3`. Checkbox: `w-4 h-4 rounded border-slate-300`.

### Inside-cell patterns

- **Avatar + primary + secondary** (name + email / title / subtitle):
  ```
  <div className="flex items-center gap-3">
    <Avatar />
    <div>
      <p className="font-semibold text-slate-900 leading-tight">{primary}</p>
      <p className="text-xs text-slate-500 leading-tight mt-0.5">{secondary}</p>
    </div>
  </div>
  ```
- **Tag pill** (department, category, location): `inline-flex items-center px-2.5 py-1 rounded-md bg-slate-100 text-slate-700 text-xs font-medium`.
- **Status pill** (dot + label): `inline-flex items-center gap-1.5 px-2.5 py-1 rounded-md text-xs font-medium` with `<span className="w-1.5 h-1.5 rounded-full bg-{color}-500" />` inside. Status bgs:
  - active / success → `bg-emerald-50 text-emerald-700`
  - pending / warning → `bg-amber-50 text-amber-700`
  - onboarding / info → `bg-blue-50 text-blue-700` or `bg-violet-50 text-violet-700`
  - danger → `bg-rose-50 text-rose-700`
  - inactive / terminated → `bg-slate-100 text-slate-600`
- **Row overflow button**: `w-8 h-8 rounded-md text-slate-400 hover:text-slate-700 hover:bg-slate-100 inline-flex items-center justify-center` wrapping `<MoreHorizontal className="w-4 h-4" />`.

## Avatars

Always **solid fill + white text** (never pastel-with-dark-text):

```
w-10 h-10 rounded-full flex items-center justify-center text-white font-semibold
```

- Sizing: `w-10 h-10` in tables; `w-9 h-9` or `w-8 h-8` for compact lists / nested rows.
- **Color per avatar** — rotate a 10-color palette hashed by the row's stable `id` so the color stays consistent across renders:
  ```
  const AVATAR_PALETTE = [
    "bg-blue-500", "bg-violet-500", "bg-teal-500", "bg-amber-500", "bg-rose-500",
    "bg-emerald-500", "bg-indigo-500", "bg-pink-500", "bg-orange-500", "bg-cyan-500",
  ];
  ```
- **System / non-person avatars**: `bg-slate-400` with white initials (e.g. "SY").
- **Profile photos**: render `<img className="w-10 h-10 rounded-full object-cover">` with `onError` → fall back to the initials avatar on load failure.

## KPI / Stat Cards

Row of 4 equal cards above a table or below a banner. Each card:

```
rounded-2xl border border-slate-200/70 bg-white px-5 pt-5 pb-5
shadow-[0_1px_2px_rgba(15,23,42,0.04),0_1px_3px_rgba(15,23,42,0.05)]
flex flex-col gap-7
```

Content:
1. Top row: icon well (left) + trend pill (right).
2. Bottom: big number (`text-3xl font-bold tracking-tight text-slate-900 leading-none`) + label (`text-sm text-slate-500 mt-2`).

- **Icon well** (uniform across all KPIs on a page): `w-10 h-10 rounded-xl bg-blue-50 flex items-center justify-center` with icon `text-blue-600 w-[18px] h-[18px]`. **Do not use per-card varied colors** (no rainbow of emerald/violet/amber/rose for KPIs on the same page) — keep the well color uniform, let the numbers do the differentiating.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kantecalebdesigns/sabihr](https://github.com/kantecalebdesigns/sabihr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
