---
trigger: always_on
description: Drag-and-drop framework for React Native (iOS, Android, Web). v1.0.0 — major rewrite.
---

# react-native-drax

Drag-and-drop framework for React Native (iOS, Android, Web). v1.0.0 — major rewrite.

## Architecture

- **UI-thread-first**: spatial index worklet for hit-testing, SharedValues split by update frequency. We need to use the UI thread as much as possible.
- **Reanimated 4 + Gesture Handler 3** (beta)
- Single `HoverLayer`, per-view gesture handlers
- Latest React features.


## Sortable Architecture

- `useSortableList` hook — list-agnostic reorder state (works with FlatList, FlashList, LegendList, etc.)
- `SortableContainer` — monitoring wrapper using `DraxView isParent`, supports `renderDropIndicator` prop
- `SortableItem` — per-item wrapper with shift animation
- `DraxList` — list-agnostic convenience wrapper (accepts `component` prop for FlatList, FlashList, LegendList, etc.)
- Map-based measurements (keyed by item key) instead of array-indexed
- Supports insert + swap reorder strategies
- Drop indicator support: `SortableContainer` tracks target position via SharedValues, renders indicator at insertion point
- **Data ownership**: Library commits reorders internally via `commitReorder`. `onReorder` is a notification — parent stores data but library already committed it. When parent echoes data back, useLayoutEffect detects the match and skips (no double-render).

### Animation Customization

- `animationConfig` prop on `useSortableList` / `DraxList`
- Presets: `'default'` (200ms timing), `'spring'` (spring physics), `'gentle'` (soft spring), `'snappy'` (fast spring), `'none'` (instant)
- Custom: `{ shiftDuration, useSpring, springDamping, springStiffness, springMass }`
- Reduced motion: `useReducedMotion()` from Reanimated — skips all shift animations automatically
- Snap-back animation: fully configurable via `animateSnap`, `snapDelay`, `snapDuration`, `snapAnimator` props

### Accessibility

- `SortableItem` auto-generates `accessibilityLabel` ("Item N of M") and `accessibilityHint` ("Long press to drag and reorder")
- `accessibilityRole="adjustable"` for screen readers
- Custom labels override defaults via `accessibilityLabel` / `accessibilityHint` props on `SortableItem`
- `useReducedMotion()` support — all shift animations respect device accessibility settings

### List Agnosticism

The composable API (`useSortableList` + `SortableContainer` + `SortableItem`) is deliberately list-agnostic:
- Works with any list component: FlatList, FlashList, LegendList, ScrollView
- The hook manages reorder state; the container monitors drags; the item wraps each cell
- `DraxList` is a convenience wrapper that accepts a `component` prop (defaults to FlatList)
- For FlashList/LegendList: pass as `component` prop to `DraxList`, or use the composable API directly

### Mixed-Size Grid (Non-Uniform Spans)

- `getItemSpan` prop on `useSortableList` — returns `{ colSpan, rowSpan }` per item
- `packGrid` utility — bin-packing algorithm placing items left-to-right, top-to-bottom into a 2D occupancy grid
- Grid geometry (cell size + gaps) derived automatically from item measurements
- `computeShiftsForOrder` uses `packGrid` to compute target positions for non-uniform items
- `getSlotFromPosition` maps finger position to grid cell, then to display index via cell→owner map
- `getSnapbackTarget` packs the pending order to find the dragged item's target position
- Rendering: user provides absolute positioning (ScrollView + absolute items); shifts handle reorder
- `packGrid` exported for users to compute grid positions in their render function
- Example: `example/screens/mixed-grid.tsx` — 4-column grid with 1×1, 2×1, 1×2, and 2×2 items

## Cross-Container Sortable (Board)

- `useSortableBoard` hook — board-level coordinator for cross-container transfers
- `SortableBoardContainer` — monitoring wrapper providing board context
- `SortableBoardContext` — auto-registration context for column `SortableContainer`s
- Each column independently uses `useSortableList` + `SortableContainer` + `SortableItem`
- Phantom slot mechanism: target column reserves virtual space at insertion point via `setPhantomSlot`
- Source column ejects dragged item from pending order via `ejectDraggedItem`
- Position-based column detection: board checks hover absolute position against column bounds
- Transfer finalization: clears all committed state on source (forces useLayoutEffect external data path), clears phantom on target, fires `onTransfer`, hover covers transition until both columns re-render with correct data
- No ghost shifts, no effectiveData bypass — both columns reset naturally via useLayoutEffect when parent updates data
- `SortableContainer` has minimal board awareness: auto-registration + finalizeDrag delegation + drag end guards

## Drag Handles

- `DraxView` accepts `dragHandle` prop — when true, the gesture is NOT attached to the view's GestureDetector
- `DraxHandle` component wraps the touchable area and receives the gesture via `DraxHandleContext`
- Only touches on the `DraxHandle` area start a drag; the rest of the view scrolls normally
- Works with `SortableItem` — just pass `dragHandle` prop and nest a `DraxHandle` inside

## Drop Zone Acceptance

- `dynamicReceptiveCallback` — conditional acceptance with full context (targetId, measurements, draggedId, draggedPayload)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nuclearpasta/react-native-drax](https://github.com/nuclearpasta/react-native-drax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
