---
trigger: always_on
description: > 给 AI Agent / LLM 编排用的浓缩版项目信息。
---

# AGENTS.md — react-zmage

> 给 AI Agent / LLM 编排用的浓缩版项目信息。
> 人类完整文档见 [README.md](./README.md)。

## Works

如果你发现工作区中有额外变更, 始终假定是另一个 AI Agent 在当前分支中同步工作, 你需要做的是优先处理重突 (如果有) 并遵循工作互不干涉原则

如果非用户主动要求, 不允许在 worktree 上执行改动。

## What this package does (1 sentence)

A React component that turns any `<img>` into a fullscreen-zoomable, multi-image,
keyboard-navigable image viewer. Drop-in replacement for `<img>`.

## Public API surface (treat as contract)

```ts
// Default + statics
import Zmage from 'react-zmage'                  // browser / bundler entry
import Zmage from 'react-zmage/ssr'              // SSR / RSC entry
import 'react-zmage/style.css'                   // required for visuals

// Types (also exported from the same module)
import type {
  BaseType,                  // union of all props (use this when typing config objects)
  ReactZmageComponent,       // typeof Zmage
  Set,                       // shape of items in `set` prop
  Preset,                    // 'desktop' | 'mobile' | 'auto' (auto = matchMedia-driven)
  ControllerSet, ControllerPlacement, ControllerOverlayLayout, ControllerLayoutTargets,
  ControllerLayoutTarget, ControllerLayoutInset, ControllerLayoutInsetValue, ControllerRender, ControllerRenderState,
  ControllerRenderActions, ControllerRenderSlots, HotKey, Animate, AnimateFlip, AnimateCoverOptions,
  GestureSet, GestureSwipeOptions, GestureDragExitOptions, GestureWheelZoomOptions,
  GesturePinchZoomOptions, GestureDoubleTapZoomOptions, GestureTouchAction,
} from 'react-zmage'

// Three usage modes
<Zmage src="..." />                                  // 1. Component
Zmage.browsing(props): () => void                    // 2. Imperative; returns destructor
<Zmage.Wrapper>{children}</Zmage.Wrapper>            // 3. Auto-attach to <img> in children

// Ref forwarding
const ref = useRef<HTMLImageElement>(null)
<Zmage src="..." ref={ref} />                        // ref points to the cover <img>
```

## Required peer deps

- `react: >=16.8.0 <20`
- `react-dom: >=16.8.0 <20`

The library auto-detects React 18+ at runtime and uses `react-dom/client.createRoot`
when available, falling back to legacy `ReactDOM.render` otherwise. No consumer
configuration needed.

## Minimum reproducible example

```tsx
import { useState } from 'react'
import Zmage from 'react-zmage'
import 'react-zmage/style.css'

export default function Demo() {
  const [open, setOpen] = useState(false)
  return (
    <>
      <Zmage
        src="https://example.com/photo.jpg"
        alt="example"
        browsing={open}
        onBrowsing={setOpen}
        set={[
          { src: 'https://example.com/01.jpg', alt: 'p1' },
          { src: 'https://example.com/02.jpg', alt: 'p2' },
        ]}
      />
    </>
  )
}
```

## Prop quick reference

Single `BaseType` covers all. Grouped here by purpose:

| Group | Props | Notes |
|---|---|---|
| Data | `src`, `alt`, `caption`, `set`, `defaultPage` | `set` enables multi-image mode. `caption` is `string \| { text, style?, className? }` — renders below the image; per-set override via `set[i].caption`. In Wrapper mode, child `<img>` nodes provide `src` / `alt`; top-level `set` may define a shared gallery, and clicked `img.src` opens the matching `set[i]` before falling back to `defaultPage`. Without `set`, `data-zmage-caption` or nearest `figcaption` may provide caption. |
| Preset | `preset: 'desktop' \| 'mobile' \| 'auto'` | defaults to `'auto'` when omitted; drives default `controller` / `hotKey` / `animate` / `gesture` plus preset-aware viewer spacing. `'auto'` resolves via `matchMedia('(pointer: coarse) and (hover: none)')` on the client; SSR falls back to desktop |
| Controlled | `browsing` | omit for self-managed; pair with `onBrowsing` if set. Does not control `<Zmage.Wrapper>` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Caldis/react-zmage](https://github.com/Caldis/react-zmage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
