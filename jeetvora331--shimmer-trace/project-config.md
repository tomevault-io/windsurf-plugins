---
trigger: always_on
description: Use when: component renders fine with real data, just need skeleton while loading.
---

# shimmer-trace — Agent & LLM Reference

> This file is written for AI agents, LLMs, and prompt engineers.
> It gives you everything needed to correctly generate, debug, and reason about shimmer-trace usage.
> Human-readable README is at `README.md`.

---

## What This Library Does (One Paragraph)

`shimmer-trace` is a React skeleton loading library. It renders your real component invisibly, walks the live DOM to find every visible leaf element (headings, paragraphs, images, inputs, buttons, etc.), measures each one's exact position and size via `getBoundingClientRect()`, then paints an absolutely-positioned shimmer overlay on top of those measured rects. The skeleton is therefore pixel-perfect and auto-updates via `ResizeObserver`. You never write a manual skeleton. You wrap your component and pass `loading={true}`.

---

## Install

```bash
npm install shimmer-trace
```

**Peer deps required:** `react >= 18.0.0`, `react-dom >= 18.0.0`

---

## Public API — Full Export List

```ts
// Components
import { Shimmer } from 'shimmer-trace'; // main component
import { ShimmerSuspense } from 'shimmer-trace'; // Suspense boundary with auto-skeleton
import { createShimmer } from 'shimmer-trace'; // factory — bakes config into component

// Hooks
import { useIsShimmering } from 'shimmer-trace'; // true when inside ShimmerSuspense fallback
import { useShimmerContext } from 'shimmer-trace'; // raw context value — advanced use only

// Raw context (rarely needed — only if you build a custom Master/Reporter outside <Shimmer>)
import { ShimmerContext } from 'shimmer-trace';

// Types
import type {
  ShimmerProps,
  ShimmerConfig,
  ShimmerRect,
  AnimationType,
  ShimmerSuspenseProps,
} from 'shimmer-trace';
```

---

## Types Reference

### `AnimationType`

```ts
type AnimationType = 'wave' | 'pulse' | 'shine' | 'glow' | 'gradient';
```

| Value | Behaviour |
|---|---|
| `wave` | Horizontal gradient sweep left→right across full container width. Default. |
| `pulse` | Opacity oscillates 0.4→1→0.4. No movement. |
| `shine` | Diagonal sweep (115°) with skew — more premium feel than wave. |
| `glow` | Brightness oscillates 1→1.35→1 via CSS filter. |
| `gradient` | Block background itself animates as sliding gradient (no child layer). |

### `ShimmerConfig`

All fields optional. Used by `Shimmer` props, `createShimmer`, and `ShimmerSuspense`.

```ts
interface ShimmerConfig {
  animation?:          AnimationType;  // default: 'wave'
  baseColor?:          string;         // default: '#e0e0e0'  (CSS color, any format)
  highlightColor?:     string;         // default: '#f5f5f5'  (CSS color, any format)
  speed?:              number;         // default: 1.5        (seconds, float)
  borderRadius?:       string;         // default: ''         (CSS value e.g. '8px'. Empty = auto-detect)
  preserveBackground?: boolean;        // default: true
}
```

**`preserveBackground` explained:**
- `true` (default): Master container stays visible. CSS rules hide text (`color:transparent`) and media (`opacity:0`) on leaf tags while keeping div backgrounds, borders, padding visible.
- `false`: Legacy mode. Master container gets `visibility:hidden`. Everything hidden. Overlay punches through with `visibility:visible`.

### `ShimmerProps`

Extends `ShimmerConfig`. All `ShimmerConfig` fields inherit plus:

```ts
interface ShimmerProps extends ShimmerConfig {
  loading?:           boolean;                    // default: false
  children:           ReactNode;                  // required
  dummyLength?:       number;                     // clone count for list mode
  dummyData?:         Record<string, any>;        // props merged into children while loading
  as?:                React.ComponentType<any>;   // component type for skeleton shape
  stopPropagation?:   boolean;                    // force Master even when nested
  className?:         string;                     // applied to Master container div
  style?:             React.CSSProperties;        // merged into Master container div
}
```

### `ShimmerSuspenseProps`

```ts
interface ShimmerSuspenseProps extends ShimmerConfig {
  children:   ReactNode;
  template?:  ReactNode;  // explicit skeleton shape; if omitted uses useIsShimmering pattern
}
```

### `ShimmerRect` (internal, rarely needed by consumers)

```ts
interface ShimmerRect {
  x:            number;  // left offset from Master container
  y:            number;  // top offset from Master container
  width:        number;
  height:       number;
  borderRadius: string;
}
```

---

## Architecture — How It Works Internally

```
<Shimmer loading={true}>           ← MasterShimmer
  children render hidden           ← visibility:hidden OR color:transparent
  useTrace() walks DOM             ← collectTraceableElements → getBoundingClientRect
  ResizeObserver re-traces         ← on container resize
  <ShimmerOverlay rects={...} />   ← absolutely positioned, z-index:1
    one <div> per traced rect      ← base color + optional SweepLayer child
    SweepLayer spans container     ← gradient child, full container width, synced wave

  <Shimmer> nested inside          ← ReporterShimmer (auto-detected via context)
    useTrace() walks own DOM       ← measures relative to Master container

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeetvora331/shimmer-trace](https://github.com/jeetvora331/shimmer-trace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
