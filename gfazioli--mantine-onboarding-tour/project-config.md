---
trigger: always_on
description: `@gfazioli/mantine-onboarding-tour` — A Mantine component library for building guided onboarding tours with focus-reveal overlays, cutout highlights, and step-by-step popover navigation.
---

# CLAUDE.md

## Project
`@gfazioli/mantine-onboarding-tour` — A Mantine component library for building guided onboarding tours with focus-reveal overlays, cutout highlights, and step-by-step popover navigation.

## Commands
| Command | Purpose |
|---------|---------|
| `yarn build` | Build the npm package via Rollup |
| `yarn dev` | Start the Next.js docs dev server (port 9281) |
| `yarn test` | Full test suite (syncpack + oxfmt + typecheck + lint + jest) |
| `yarn jest` | Run only Jest unit tests |
| `yarn docgen` | Generate component API docs (docgen.json) |
| `yarn docs:build` | Build the Next.js docs site for production |
| `yarn docs:deploy` | Build and deploy docs to GitHub Pages |
| `yarn lint` | Run oxlint + Stylelint |
| `yarn format:write` | Format all files with oxfmt |
| `yarn storybook` | Start Storybook dev server |
| `yarn clean` | Remove build artifacts |
| `yarn release:patch` | Bump patch version and deploy docs |
| `diny yolo` | AI-assisted commit (stage all, generate message, commit + push) |

> **Important**: After changing the public API (props, types, exports), always run `yarn clean && yarn build` before `yarn test`, because `yarn docgen` needs the fresh build output.

## Architecture

### Workspace Layout
Yarn workspaces monorepo with two workspaces: `package/` (npm package) and `docs/` (Next.js 15 documentation site).

### Package Source (`package/src/`)

The library has two main use cases:
1. **OnboardingTour** — step-by-step guided tour with popover navigation
2. **FocusReveal** — standalone focus/highlight for individual elements (also used internally by OnboardingTour)

```
OnboardingTour.tsx                — Main wrapper; factory() component, recursively clones children matching data-onboarding-tour-id
OnboardingTour.context.ts         — OnboardingTourContext provider
OnboardingTour.module.css         — Styles for root overlay and tour
├── OnboardingTourFocusReveal/    — Overlay + popover + scroll-into-view + animation per element (plain function component)
│   ├── focus-reveal-modes.ts     — 12 CSS animation mode definitions
│   └── OnboardingTourFocusReveal.module.css
├── OnboardingTourFocusRevealGroup/ — Shared overlay for multiple FocusReveals (scroll-reveal pattern)
│   └── OnboardingTourFocusRevealGroup.context.ts
├── OnboardingTourPopoverContent/ — Popover body: header, title, content, footer, nav buttons, stepper (factory() component)
│   └── OnboardingTourPopoverContent.module.css
├── OnboardingTourTarget/         — Declarative target for elements outside the OnboardingTour tree (plain function component)
└── hooks/
    ├── use-onboarding-tour/      — Internal hook: currentStepIndex, pendingStepIndex, tour controller
    ├── use-cutout-rect/          — Measures focused element, builds SVG clip-path with cutout hole
    └── use-scroll-into-view/     — Custom fork (not @mantine/hooks): external scrollableRef, RAF easing, cancel on user scroll
        └── utils/                — easeInOutQuad, getRelativePosition, getScrollStart, setScrollParam
```

Compound component registration: `OnboardingTour.FocusReveal`, `OnboardingTour.PopoverContent`, `OnboardingTour.Target`, plus `OnboardingTour.FocusReveal.Group`.

### Build Pipeline
Rollup bundles to dual ESM (`dist/esm/`) and CJS (`dist/cjs/`) with `'use client'` banner. CSS modules are hashed with `hash-css-selector` (prefix `me`). TypeScript declarations via `rollup-plugin-dts`. CSS is split into `styles.css` and `styles.layer.css` (layered version).

### Docs (`docs/`)
Next.js 15 static export. Demos in `docs/demos/` export a `Wrapper` function + metadata object compatible with `@mantinex/demo`. Demo categories:
- **OnboardingTour.demo.***: configurator, cutout, target, customStepper, customPopoverContent, customEntry, wrapTitle, onboardingTourStep, onboardingTourStepFocusReveal, onboardingProps, targetFocusReveal
- **Full-page demos** (docs/pages/): `demo.tsx` (main tour), `responsive.tsx` (responsive positioning)
- **FocusReveal.demo.***: configurator, cycle, group, group-props, overlay, popover, popoverProps, reveal, scrollContainer, focusMode, paper, uncontrolled, disableTargetInteraction, cycleDescription

## Component Details

### Data Flow

```
started=true → useEffect → startTour() → setCurrentStepIndex(0)
  → OnboardingTour renders persistent overlay (fixed, full-screen, CSS clip-path cutout)
  → wrapChildren() recursively walks React children tree
    → matches data-onboarding-tour-id to tour[currentStepIndex].id
    → wraps matched child in <FocusReveal focused={true} withOverlay={false} popoverContent={<PopoverContent/>}>
      → useScrollIntoView scrolls element into viewport (custom fork of Mantine's hook)
      → useInViewport detects visibility → sets realFocused=true
      → opens Popover (withinPortal=true) with step content
  → User clicks Next → nextStep():
      Phase 1: setPendingStepIndex(n+1) → selectedStepId=undefined → current popover closes
      Phase 2: useEffect → setCurrentStepIndex(n+1) → new popover opens
      (persistent overlay stays visible throughout, CSS-transitions overlayProps changes)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gfazioli) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
