---
trigger: always_on
description: Architecture and implementation details for AI agents and contributors.
---

# react-in-viewport — Agent Reference

Architecture and implementation details for AI agents and contributors.

---

## Overview

`react-in-viewport` exposes React components/elements to the
[Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API).
It ships two public APIs that share the same core hook:

| Export | Type | Use case |
|---|---|---|
| `handleViewport` | Higher-Order Component (HOC) | Wrap any class or functional component |
| `useInViewport` | Hook | Compose directly in functional components |

---

## Package Structure

```
src/
  index.ts                  # Public API re-exports + customProps constant
  lib/
    useInViewport.tsx       # Core hook — IntersectionObserver + MutationObserver
    handleViewport.tsx      # HOC that wraps any component with useInViewport
    types.ts                # Shared TypeScript types
    constants.ts            # Shared defaults (defaultOptions, defaultConfig, noop)
    utils.ts                # omit() — strips injected props before native element spread
  stories/
    chapters/               # Storybook story files (one per use-case)
    common/                 # Shared story components (Block, Image, Iframe, Section, etc.)
  __tests__/
    index.tsx               # Jest tests (jsdom + mocked IntersectionObserver)
  setupTests.js             # Polyfills IntersectionObserver for jsdom
```

---

## Core Hook: `useInViewport`

```tsx
const { inViewport, enterCount, leaveCount } = useInViewport(
  target,   // RefObject<HTMLElement | null>
  options,  // IntersectionObserverInit — passed to IntersectionObserver constructor
  config,   // { disconnectOnLeave?: boolean }
  props,    // { onEnterViewport?: () => void, onLeaveViewport?: () => void }
);
```

### Two-effect design

**Effect 1 — MutationObserver (deferred-ref handling)**

Solves a race condition where the ref (`target.current`) passed in is `null` at
mount time (e.g. a ref created outside and passed down). A `MutationObserver`
watches `document.body` for `childList` changes. When it detects a mutation and
`target.current` is now non-null, it sets `isTargetReady = true` and
immediately disconnects itself.

```
isTargetReady: false → (MutationObserver fires) → true
```

For `handleViewport`, the ref is always available at mount, so this observer
fires and disconnects immediately — zero ongoing cost.

**Effect 2 — IntersectionObserver (main viewport tracking)**

Depends on `[isTargetReady, options, config, onEnterViewport, onLeaveViewport]`.
Runs when `isTargetReady` becomes true (either on initial mount if the ref is
already set, or after the MutationObserver fires). Creates the
`IntersectionObserver`, starts observing the target node, and cleans up
(unobserve + disconnect) on dep change or unmount.

> **Important**: The effect's cleanup always calls `disconnect()` — this happens
> both on unmount and when deps change (e.g. the `isTargetReady` false→true
> transition). Tests that assert on `mockDisconnect` must clear the mock after
> `render()` to isolate intersection-specific disconnect calls.

### State model

All viewport state is stored in `useRef` (not `useState`) to avoid
double-render on each intersection event. A single `forceUpdate` boolean state
is used solely to trigger a re-render when the viewport state changes.

```ts
inViewportRef    useRef<boolean>(false)  // current in/out state
intersected      useRef<boolean>(false)  // guards duplicate enter/leave
enterCountRef    useRef<number>(0)
leaveCountRef    useRef<number>(0)
```

### `disconnectOnLeave`

When `config.disconnectOnLeave = true`, the `IntersectionObserver` is
disconnected as soon as the element leaves viewport. Subsequent enter/leave
events will not fire. This is a performance optimisation for "load once on
enter" patterns (e.g. lazy images).

---

## HOC: `handleViewport`

```tsx
const WrappedComponent = handleViewport(MyComponent, options?, config?);
// Usage:
<WrappedComponent onEnterViewport={fn} onLeaveViewport={fn} />
```

`handleViewport` injects the following props into `MyComponent`:

| Prop | Type | Description |
|---|---|---|
| `inViewport` | `boolean` | Whether the element is currently visible |
| `enterCount` | `number` | Number of times it has entered viewport |
| `leaveCount` | `number` | Number of times it has left viewport |
| `forwardedRef` | `RefObject<TElement>` | Must be attached to a DOM element |

### Critical: always attach `forwardedRef`

The wrapped component **must** attach `forwardedRef` to a DOM node. Without
it, `target.current` is always `null`, `isTargetReady` never becomes `true`,
and the `IntersectionObserver` is never created — the component will always
render as "not in viewport".

```tsx
// ✅ correct
const MyComponent = ({ inViewport, forwardedRef }: InjectedViewportProps) => (
  <div ref={forwardedRef}>{inViewport ? 'visible' : 'hidden'}</div>
);

// ❌ forwardedRef not attached — observer never fires
const MyComponent = ({ inViewport }: InjectedViewportProps) => (
  <div>{inViewport ? 'visible' : 'hidden'}</div>
);
```

### Forwarding props to native elements

The HOC injects `inViewport`, `enterCount`, and `leaveCount` — these are not
valid HTML attributes and must be stripped before spreading to a native element.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roderickhsiao/react-in-viewport](https://github.com/roderickhsiao/react-in-viewport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
