---
trigger: always_on
description: | Layer | Technology |
---

# Airmedy Project Mandates

## Tech Stack

| Layer | Technology |
|---|---|
| Desktop framework | Wails v3 (Go + Vue 3) |
| Monorepo | pnpm workspaces + Turbo |
| DI | Uber FX |
| Frontend state | Pinia 3 |
| i18n | vue-i18n |
| UI components | Radix Vue, TailwindCSS v4 (`@airmedy/ui`) |
| Database | SQLite (sqlx + golang-migrate) |
| Search | Bleve v2 |
| File watching | fsnotify |
| Audio (macOS) | SFBAudioEngine (cgo) |
| Audio (Win/Linux) | miniaudio (C) |
| Metadata | TagLib + FFmpeg fallback |
| Logging | log/slog + lumberjack |

---

## 1. Performance

- Track lists/grids MUST use `vue-virtual-scroller` (56px row height, or 36px in compact/collapsed mode).
- Heavy tasks (metadata extraction, directory scan, search indexing) MUST run in goroutines with `context.Context` cancellation.
- SQLite: use transactions for bulk writes; maintain appropriate indexes.

---

## 2. UI Design System

### Glass-Morphism Rules

- Surfaces: `background: var(--bg-glass); backdrop-filter: blur(30px)`.
- Borders: `1px solid var(--border-glass)`. Never plain `border` without explicit opacity.
- Row hover: `hover:bg-white/[0.04]`. Never `hover:bg-accent/50`.
- Button inactive/active: opacity variation (`text-white/40 → text-white/70`), not color switch.
- Play button: white circle + black icon (`bg-white text-black`).
- Secondary/metadata text: `text-white/40` or `text-white/30`. Never `text-muted-foreground` on dark translucent backgrounds.
- Cards: `border-radius: 12px`, hover `scale(1.02)` + border brightness, shadow `0 10px 15px -3px rgba(0,0,0,0.4)`.
- Icons: **Lucide Vue only**. No Phosphor or others.
- Progress bars: 4px, expands 6px on hover, white thumb.
- Transitions: `all 0.3s cubic-bezier(0.4, 0, 0.2, 1)`. Theme color shifts: `1.5s ease-in-out`.

### TailwindCSS v4

- Tokens defined via `@theme` directive in global CSS — **not** `tailwind.config.js extend.colors`.
- CSS custom properties are the primary theming mechanism.
- New design tokens go under `@theme` only.

### Mandatory CSS Variables

| Variable | Dark value | Black (OLED) value | Purpose |
|---|---|---|---|
| `--bg-main` | `#18181B` | `#0A0A0A` | Page background |
| `--bg-glass` | `rgba(35,35,38,0.6)` | `rgba(25,25,25,0.6)` | Glass surfaces |
| `--bg-glass-elevated` | `rgba(55,55,60,0.4)` | `rgba(45,45,45,0.4)` | Elevated glass |
| --border-glass | `rgba(255,255,255,0.1)` | same | Glass borders |
| `--primary` | `#E11D48` | same | Primary action |
| `--dynamic-primary` | extracted vibrant | same | Per-track, runtime |
| `--dynamic-surface` | extracted dominant 10–20% opacity | same | Per-track, runtime |
| `--dynamic-glow` | extracted vibrant 40% opacity | same | Per-track, runtime |

Themes: `system`, `light`, `dark` (gray), `black` (pure black for OLED). `black` applies `.dark` + `.black` CSS classes; `.black` only overrides bg vars.

### Layout Constraints

- Sidebar: `240px` width, `30px` blur.
- Player bar: `80px` height, `30px` blur, `1px solid var(--border-glass)` top border, fixed bottom.

### Typography Scale

- H1 Hero: 32px Bold, tracking -0.02em
- H2 Section: 20px Semibold
- Body: 14px Medium
- Metadata: 12px Regular, 60% opacity

### UI Primitive Rules

- Sliders/progress: `import { Slider } from '@airmedy/ui'` — never `<input type="range">`.
- Text inputs: `import { Input } from '@airmedy/ui'` — never raw `<input type="text">`.
- New shared primitives (no app-domain knowledge) → `packages/ui/src/`, export from `packages/ui/src/index.ts`.
- App-specific components (use stores/bindings/router) → `frontend/src/components/`.

### UI Implementation Checklist

- [ ] Glass blur or subtle border for depth?
- [ ] `--dynamic-*` colors verified for WCAG contrast?
- [ ] All interactions smooth — no abrupt state changes?
- [ ] Icons are Lucide Vue only?
- [ ] Cards scale 2% on hover with brighter border?
- [ ] New tokens via `@theme` directive, not config `extend`?

### Package Manager

**pnpm** only. Never npm or yarn.

---

## 2.1 Internationalization

- **12 locales:** `de en es fr it ja ko pt ru th vi zh` — all in `frontend/src/locales/`.
- Never hardcode strings. Use `$t()` in templates, `t()` from `useI18n()` in scripts.
- Never write `t('key', 'Fallback')`. Keys must exist in locale files.
- When adding a key: add to **all 12 files simultaneously** (English value + translated/placeholder in other 11), then use in component.

---

## 2.5 Frontend Component Architecture

| Type | Location | Rules |
|---|---|---|
| UI Primitives | `packages/ui/src/` (`@airmedy/ui`) | No `stores/`, `bindings/`, or `router` imports. Stateless. Shared across workspaces. |
| Feature Components | `frontend/src/components/` | May import stores/bindings. Reused across views. |
| Views/Pages | `frontend/src/views/` | Orchestrates feature components, handles page fetching. |

- Extract to composable/component when pattern appears in 3+ places.
- Prefer `<slot />` over excessive props.
- Split components >300 lines.
- Shared logic → `lib/utils.ts` or `composables/`.
- All components: `<script setup lang="ts">`, typed `defineProps<{}>()` and `defineEmits<{}>()`.
- Every non-trivial feature component needs a `.spec.ts` (Vitest + `@vue/test-utils`).

---

## 2.6 Frontend Memory Management

### Wails Events (`Events.On`)

- `Events.On()` returns `() => void` — **always store the off-function**.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [misa198/airmedy](https://github.com/misa198/airmedy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
