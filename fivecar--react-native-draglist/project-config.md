---
trigger: always_on
description: - `npm run build` — builds via microbundle-crl (`--no-compress --format modern,cjs`) into `dist/`
---

# CLAUDE.md

## Build Commands

- `npm run build` — builds via microbundle-crl (`--no-compress --format modern,cjs`) into `dist/`
- `npm run release` — releases via release-it (with conventional changelog and GitHub release)
- `npm run prepare` — runs build automatically on `npm install`
- `npm test` — runs the Jest regression suite in `src/__tests__/` (responder termination, mid-drag data changes, key stability, mirrored/RTL layouts). No linting configured.

## Development Workflow

After making changes to source files, rebuild and test in the example app:

```sh
npm run build
cd example
npm i ..          # reinstalls the freshly built dist/ into the example
npm run android   # or npm run ios
```

All validation is manual — test on **both iOS and Android**. Key test cases:
- Drag reorder in short and long/scrolling lists
- Auto-scroll when dragging beyond list bounds
- Drag-and-release back to original position
- "Scroll to Top" button (verifies forwardRef)
- Horizontal list dragging
- The same horizontal cases under RTL, via the example app's direction toggle. On iOS the toggle's
  reload isn't enough — relaunch natively (`xcrun simctl terminate`/`launch`) for `forceRTL` to take
  effect. Watch the printed data order under the horizontal list: the visual order mirrors, so it's
  the only way to tell a correct drop from one that landed at the mirrored index.

## Architecture

Two source files:

- **`src/index.tsx`** — `DragList` component (forwardRef wrapper around FlatList). Manages drag state via PanResponder, auto-scrolling, and reorder logic. Exports `DragListRenderItemInfo<T>` with `onDragStart`/`onDragEnd`/`isActive`. Contains `CellRendererComponent` which handles per-cell animation (slide displacement via `Animated.timing`).
- **`src/DragListContext.tsx`** — `DragListProvider` context and `useDragListContext` hook. Passes drag state (activeData, pan, panIndex, layouts) from DragList into CellRendererComponent without prop drilling.

### Key Patterns

- **Built-in Animated API + PanResponder only** — intentionally avoids react-native-reanimated. No external dependencies beyond React/React Native peer deps.
- **Layout caching in refs** — `layouts` ref stores `{ pos, extent }` per item key, used for hit-testing and displacement calculations. Axis-independent (pos/extent works for both horizontal and vertical).
- **Stable keys + static idle transforms** — item keys are stable across data changes (required for `maintainVisibleContentPosition` on Fabric, and avoids remounting every row). Cells attach Animated transform nodes ONLY while a drag is active; when idle they render a static `transform: 0`, so the commit that applies reordered data carries zeroed transforms atomically with the new layout. This is the flash-on-drop fix: async native-animated commands can never race the commit because nodes attach at value 0 and detach in commits that already specify 0. Do not reintroduce always-attached Animated transforms or generation-suffixed keys. The `layouts` cache is pruned of removed keys on each data change.
- **Drag teardown invariant** — `props.onDragBegin`/`props.onDragEnd` must always pair up. `dragEndOwedRef` tracks the debt; every teardown path (release, `onPanResponderTerminate`, mid-drag data change) settles it via `fireOwedDragEnd`. Responder termination commits the reorder at the current hover index (deliberate choice — see README caveat on gesture recognizers).
- **Auto-scroll frame loop** — dragging past an edge drives `scrollToOffset({animated: false})` from a `requestAnimationFrame` loop at a speed that ramps with how far past the edge you are. It used to step a whole item per 200ms interval with `animated: true`, which read as a series of jumps (#94). `onContentSizeChange` is destructured out of `rest` and chained (like `onScroll`/`onLayout`) because `rest` is spread last and would otherwise let a host silently replace our handler.

  **The loop, not `onScroll`, is the authority on where the list is mid-drag.** This is the one invariant to keep in mind when touching any of this. `onScroll` reports lag a frame or more *and arrive unevenly*, so the loop integrates its own offset (`autoScrollOffsetRef` in flow space, `autoScrollScrollPosRef` its cartesian twin) and `updateRendering` draws against that via `effectiveScrollPos()`. Drawing against `scrollPos` instead jitters the dragged item against smoothly moving content, and leaves the loop's last command before an end-of-list pin undrawn so a release reorders to a stale slot.

  The invariant is that the loop's offset equals the list's real offset for as long as a drag is live. It has exactly three dependencies, and *every* bug found in this feature so far has been one of them breaking. Check new work against this list rather than rediscovering them:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fivecar/react-native-draglist](https://github.com/fivecar/react-native-draglist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
