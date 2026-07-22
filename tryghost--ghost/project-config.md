---
trigger: always_on
description: Canonical, rule-shaped reference for AI-assisted work on Shade and any admin app that consumes it. Storybook docs at `apps/shade/src/docs/` are the human-facing surface (visual, designer-focused). **This file is the source of truth for decisions.**
---

# Shade — Agent guide

Canonical, rule-shaped reference for AI-assisted work on Shade and any admin app that consumes it. Storybook docs at `apps/shade/src/docs/` are the human-facing surface (visual, designer-focused). **This file is the source of truth for decisions.**

## Core assumptions

- **Shade is the default source for Ghost Admin UI.** Reach for it first. If a usable primitive, component, recipe, or pattern exists, use it.
- **Shade is admin-only.** Don't generate install instructions, stylesheet imports, or `ShadeApp` setup snippets — every admin app is already wired up.
- **Imports come from layer-specific subpaths**, never the root barrel:
  ```ts
  import {Stack, Inline, Box, Grid, Container, Text} from '@tryghost/shade/primitives';
  import {Button, Input, Dialog} from '@tryghost/shade/components';
  import {PageHeader, KpiCard, Filters} from '@tryghost/shade/patterns';
  import {ListPage} from '@tryghost/shade/page-templates';
  import {PostShareModal} from '@tryghost/shade/posts-stats';
  import {cn} from '@tryghost/shade/utils';
  import {ShadeApp} from '@tryghost/shade/app';
  ```
- Inside Shade itself, use the `@/` alias for cross-file imports.

## The five layers

| Layer | Path | Use when | Examples |
|---|---|---|---|
| **Tokens** | `theme-variables.css`, `tailwind.theme.css` | You need a colour, size, duration, radius | `--background`, `--text-base`, `--radius-md` |
| **Primitives** | `src/components/primitives/` | You need layout structure | `Stack`, `Inline`, `Box`, `Grid`, `Container`, `Text` |
| **Components** | `src/components/ui/` | You need a generic, accessible UI control | `Button`, `Input`, `Dialog`, `Tabs`, `Card`, `DropdownMenu` |
| **Recipes** | `src/components/ui/<name>.ts` | Several components share the same visual rule (chrome, focus, density) | `inputSurface` |
| **Patterns** | `src/components/patterns/` | The shape is product-specific and recurs across Admin | `PageHeader`, `Filters`, `KpiCard`, `GhAreaChart` |

Plus two additional barrels:

- **`page-templates/`** (`src/components/page-templates/`) — top-level page wrappers (`ListPage` today). Composes Patterns + Components + Primitives. Imported via `@tryghost/shade/page-templates`.
- **`posts-stats/`** (`src/components/posts-stats/`) — transitional layer for components shared between `apps/posts` and `apps/stats` until those merge. Don't generalise it. Imported via `@tryghost/shade/posts-stats`.

## Decision flow: where does new code go?

When building a new UI shape, walk this top-to-bottom and stop at the first match.

1. **Is it just a colour, size, radius, duration?** → **Token**. Add to `theme-variables.css` (semantic) or `tailwind.theme.css` (`@theme` raw).
2. **Is it layout-only (spacing, alignment, structure)?** → **Primitive**. Use an existing one (`Stack`, `Inline`, `Box`, `Grid`, `Container`, `Text`); only add a new one if the structural shape is genuinely novel.
3. **Is it a generic, accessible UI control with no Ghost-specific knowledge?** → **Component**. Reuse an existing one in `src/components/ui/`. Only add a new component if it doesn't exist and the rules below pass.
4. **Is it the same chrome / focus / density rule shared across ≥ 2 components?** → **Recipe**. A class-string function next to the components in `src/components/ui/`.
5. **Does it know about Ghost (KPIs, members, posts, newsletters, analytics)?** → **Pattern**.

Quick gut check: **generic name → Component; Ghost-shaped name → Pattern.** `Button` is web-y; `KpiCard` is Ghost-y.

## When to ADD to Shade vs keep local

The default is to **keep code local first**. Premature design system additions lock in the wrong API and every consumer pays when you change it.

Promote to Shade only when **all** are true:

1. **Reused at least twice in different surfaces.** Not "we might reuse this" — actual second use.
2. **It's generic.** A `<MembersTable>` that's just `<Table>` with three pre-set columns is not a Shade thing; it belongs in the app.
3. **The shape has settled.** Slots and composition have been stable across both local copies for at least one iteration cycle.
4. **It has a generic name.** `PageHeader`, `KpiCard`, `PostShareModal`. Not `MembersFilterBar` or `PostAnalyticsHero` — those name a single surface and will date.
5. **The API is slots, not props.** 3–6 named subcomponents (`.Title`, `.Actions`, `.Body`), not a `<ListPage title="..." onAdd={...} columns={...} />` prop bag.
6. **State stays with the consumer.** No `useQuery`, no routing, no app-context reads inside Shade.

Fail any of these? Keep it local. Build it again somewhere else first, then promote.

## Conventions

### File names

- Files: kebab-case (`dropdown-menu.tsx`) — matches ShadCN CLI output.
- Components: PascalCase exports (`DropdownMenu`).
- Hooks, functions, variables: camelCase.

### Component file structure

- One `<name>.tsx` per component (or compound family).
- Sibling `<name>.stories.tsx` is required.
- Use `cn()` to merge classes (`@tryghost/shade/utils` for consumers, `@/lib/utils` inside Shade).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TryGhost/Ghost](https://github.com/TryGhost/Ghost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
