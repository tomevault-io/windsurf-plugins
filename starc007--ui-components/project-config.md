---
trigger: always_on
description: React motion component library plus its docs site, distributed as copy-paste source via shadcn-compatible registry endpoints. Stack: Next.js 15 (App Router), React 19, Tailwind CSS 4, motion (framer-motion) v11, TypeScript strict, Bun, Biome. Live at beui.dev.
---

# beUI v2 — agent guide

React motion component library plus its docs site, distributed as copy-paste source via shadcn-compatible registry endpoints. Stack: Next.js 15 (App Router), React 19, Tailwind CSS 4, motion (framer-motion) v11, TypeScript strict, Bun, Biome. Live at beui.dev.

## Commands

```bash
bun install
bun run dev             # local site
bun run typecheck       # tsc --noEmit
bun run lint            # biome
bun run check:registry  # every registry component can publish its files
bun run check           # all three — run before committing
```

Prefer `typecheck` + `lint` for quick verification. Do not start the dev server or run `bun run build` unless explicitly asked.

## Layout

- `components/motion/`, `components/charts/`, and `components/agents/` — the library. One file per component; multi-file widgets get a folder (`swap/`, `button/`, `loading-states/`).
- `components/previews/` — demo per component, registered in `components/previews/index.tsx`. Previews ship through the registry too. When an interactive preview uses internal demo helpers, set the registry entry's `usageFile` to a separate public composition so the Usage tab stays copyable.
- `components/app/` — site chrome (header, hero, dock, code blocks). Not part of the library.
- `lib/registry.ts` — component catalog (slugs, files, examples). Four categories: `motion` (display name "Components", primitives), `agents` (AI and agent interface primitives), `charts` (data visualizations at `/charts`, with detail pages at `/charts/<slug>`), and `blocks` (composed widgets: swap, dynamic island, command palette, expandable action bar). Blocks emit `registry:block` shadcn items. Preview files live under `components/previews/<category>/`. `lib/registry-server.ts` builds registry items by following each file's `@/` and relative imports and bundling everything it finds. Internal imports are therefore safe and encouraged; a component that imports `@/lib/ease` ships `lib/ease.ts` with it.
- `app/r/*` — registry endpoints (shadcn items, raw source, index). beUI is listed in shadcn's official registry directory as the `@beui` namespace with URL template `https://beui.dev/r/{name}.json` — that path shape and existing install slugs are public contract; never break or rename them.
- `lib/ease.ts` — all motion tokens.
- `scripts/check-registry.ts` — validates the catalog.

## Component catalog

Before building a new component, check this list. If it exists, import it. If it doesn't, create it following the conventions below.

### Components (`motion` category — primitives)

| slug | file | what it does |
|---|---|---|
| `tilt-card` | `components/motion/tilt-card.tsx` | 3D perspective tilt on hover with cursor-tracked glare |
| `button` | `components/motion/button/` | Spring-pressed `Button` (optional `ripple` prop for a Material-style press ripple), `StatefulButton` (idle/loading/success/error), `MagneticButton` |
| `expandable-control` | `components/motion/expandable-control.tsx` | Click-to-expand `ExpandableButton` and `ExpandableChip` controls that reveal a label or trailing action with layout continuity; controlled/uncontrolled, reduced-motion safe |
| `adaptive-stepper` | `components/motion/adaptive-stepper.tsx` | Composable numeric stepper whose fixed footprint adapts at its bounds: the unavailable action exits while the value surface expands into its space; controlled/uncontrolled, keyboard focus handoff, reduced-motion safe |
| `marquee` | `components/motion/marquee.tsx` | Infinite horizontal or vertical scroll, pause-on-hover |
| `tabs` | `components/motion/tabs.tsx` | Pill, segment or underline tabs with spring layoutId indicator |
| `switch` | `components/motion/switch.tsx` | Toggle with spring-driven thumb and press feedback |
| `select` | `components/motion/select.tsx`, `select-morph.tsx` | Composable select primitives (`Select`, `SelectTrigger`, `SelectValue`, `SelectContent`, `SelectItem`); panel bouncily unfolds out of the trigger and separates (position-aware). `MorphSelect` (`select-morph.tsx`) is a shared-layout variant where the trigger grows into the panel and back |
| `combobox` | `components/motion/combobox.tsx` | Composable searchable selection primitives whose input is the trigger; the measured list springs open from the field and detaches without page reflow, with stable collision-aware placement through exit, grouped filtering, keyboard navigation that is live as soon as the list appears, a spring-gliding active row and controlled/uncontrolled state |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [starc007/ui-components](https://github.com/starc007/ui-components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
