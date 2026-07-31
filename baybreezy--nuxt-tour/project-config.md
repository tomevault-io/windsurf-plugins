---
trigger: always_on
description: This file gives an AI agent the context needed to work effectively on this repo without re-deriving everything from scratch.
---

# Agent Instructions — nuxt-tour

This file gives an AI agent the context needed to work effectively on this repo without re-deriving everything from scratch.

---

## What this repo is

**nuxt-tour** is a Nuxt module that lets developers add interactive guided tours to their apps. It ships:

- A `<VTour />` component (auto-registered, prefix configurable)
- A `useTour(name)` composable (auto-imported)
- A default stylesheet (`tour.css`) built with CSS custom properties

Package manager: **bun**. Do not use npm or yarn for installs.

---

## Essential commands

```bash
# Build the module stub (run this before anything else after a fresh clone)
bun run dev:prepare

# Start the docs dev server (docs/ is the playground for manual testing)
bun run dev

# Run all tests
bun run test

# Type-check
bun run test:types

# Lint
bun run lint

# Format
bun run fmt
```

---

## Directory layout

```
src/
  module.ts                      # Nuxt module entry — options, component + composable registration
  runtime/
    components/Tour.vue          # Main component (~300 lines)
    composables/useTour.ts       # Public composable — module-level Map registry
    utils/
      storage.ts                 # localStorage read/write/TTL/versioning/migration
      popper.ts                  # Popper.js factory + deepMerge helpers
      scroll.ts                  # scrollToTarget() wraps jump.js / scrollIntoView
    css/tour.css                 # Default styles — all CSS custom properties (--nt-*)
    types.ts                     # All exported types (TourConfig, TourStep, TourEmits, etc.)
docs/                            # Documentation site (Nuxt + Docd)
test/
  basic.test.ts                  # E2E: basic fixture
  multiStep.test.ts              # E2E: 3-step fixture
  storage.test.ts                # Unit: storage helpers (13 cases)
  composable.test.ts             # Unit: useTour composable (7 cases)
  fixtures/
    basic/                       # Single-step auto-start fixture
    multistep/                   # Three-step fixture (no auto-start)
    storage/                     # saveToLocalStorage: 'end' fixture
.github/
  workflows/ci.yml               # Lint + typecheck + test on push/PR
  workflows/release.yml          # Manual workflow_dispatch release to npm
  ISSUE_TEMPLATE/bug_report.yml
  ISSUE_TEMPLATE/feature_request.yml
CONTRIBUTING.md
AGENTS.md  ←  you are here
CLAUDE.md  →  symlink to AGENTS.md
```

---

## Module options (`nuxt.config.ts`)

```ts
tour: {
  prefix?: string          // Component name prefix. Default "V" → <VTour />
  injectCSS?: boolean      // Inject tour.css. Default true
  storagePrefix?: string   // localStorage key prefix. Default "nt"
  storageVersion?: string  // Bump to force re-show of played tours globally
}
```

---

## Types

All public types live in `src/runtime/types.ts` and are importable as:

```ts
import type { TourStep, TourConfig, TourEmits, TourStorageEntry } from "#nuxt-tour/types";
```

Key types:
- **`TourConfig`** — all component props (was `TourProps` in v0)
- **`TourStep`** — step definition
- **`ButtonConfig`** — button label + icons (was `ButtonProp` in v0)
- **`TourStorageEntry`** — localStorage schema `{ completed, status, completedAt, lastStep, version }`
- **`TourState`** — internal composable state held in the module-level Map

---

## Component — `Tour.vue`

Key decisions to know:

- **SSR safety**: every `window`/`localStorage` access is guarded by `import.meta.client`. `useScrollLock` is only called client-side.
- **Popper**: `createTourPopper()` from `utils/popper.ts` wraps Popper.js. The component calls `refreshPopper()` on each step transition.
- **Slot bindings**: all slots receive a single `slotProps` computed — do not duplicate bindings.
- **`data-hidden` attribute** controls visibility (CSS `display: none !important`). The `v-show` transition runs on top of that.
- **Backdrop**: `syncBackdrop()` is the single source of truth — call it after every step transition. The old code had duplicated logic that could leave the backdrop visible; this version does not.
- **Exposed**: `startTour`, `endTour`, `skipTour`, `nextStep`, `prevStep`, `goToStep`, `pause`, `resume`, `resetTour`, `recalculatePopper` (alias for `refreshPopper`), `isLocked`, `currentStep`, `isActive`

---

## Composable — `useTour(name, storagePrefix?)`

State is held in a module-level `Map<string, TourState>`. Both the component and external code access the same reactive object for a given `name`.

- `isPlayed` is a computed backed by `state._storageCompleted` (a reactive mirror of localStorage). **Do not read localStorage directly in computed** — Vue won't track it. Always go through `_storageCompleted`.
- When the component mounts it wires `_start`, `_end`, `_skip`, etc. onto the state. If `start()` is called before mount it sets `_pendingStart = true` and the component picks it up in `onMounted`.
- `markPlayed()` / `markUnplayed()` / `reset()` all update both localStorage and `_storageCompleted`.

---

## Storage (`utils/storage.ts`)

Schema stored as JSON under key `{storagePrefix}-{name}`:

```ts
{
  completed: boolean
  status: 'completed' | 'skipped' | null
  completedAt: string | null   // ISO date — TTL comparison

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BayBreezy/nuxt-tour](https://github.com/BayBreezy/nuxt-tour) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
