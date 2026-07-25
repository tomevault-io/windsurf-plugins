---
trigger: always_on
description: Manages focus requests with retry logic. Uses `View.postDelayed()` for 300ms retries (max 3). Uses a `volatile boolean lock` to cancel pending retries once focus is acquired. Delegates actual focus posting to `ChoreographerUtils` to ensure layout stability. Also stores a weak reference to the last focused view via `storeViewReference()` / `getStoredView()`.
---

# Android Native Layer

Java implementation of the accessibility order library. Supports both New Architecture (Fabric/TurboModules) and Old Architecture (Bridge).

## Directory Structure

```
android/src/
├── main/java/com/a11yorder/
│   ├── A11yOrderPackage.java               # TurboReactPackage — registers 5 view managers (Index, Order, PaneTitle, Lock, Card) + AnnounceModule
│   ├── core/                               # Inheritance chain (bottom → top):
│   │   ├── A11yViewGroup.java              #   Base — weak-ref first-child tracking (onChildAttached/onChildRemoved)
│   │   ├── A11yScreenReaderView.java       #   ↳ screen reader events (focused/focusChanged/descendantFocusChanged)
│   │   ├── A11yManagedFocusView.java       #   ↳ autoFocus prop, focus() command, A11yFocusProtocol
│   │   └── A11yViewOrder.java              #   ↳ A11yOrderService wiring (index/key/focusType, child linking)
│   ├── events/
│   │   ├── EventHelper.java                # Dispatch utilities for all custom events
│   │   ├── ScreenReaderFocusChangedEvent   # isFocused boolean event payload
│   │   ├── ScreenReaderFocusedEvent        # View focused (no payload)
│   │   └── ScreenReaderDescendantFocusChangedEvent  # status + nativeId payload
│   ├── modules/
│   │   └── A11yAnnounceModule.java         # TurboModule for screen reader announcements (announce/cancel/cancelAll)
│   ├── services/
│   │   ├── focus/
│   │   │   ├── A11yFocusDelegate.java      # Coordinates focus with fragment lifecycle
│   │   │   ├── A11yFocusProtocol.java      # Interface: isViewFocused()
│   │   │   └── A11yFocusService.java       # Singleton focus manager with retry logic + view ref storage
│   │   └── order/
│   │       ├── A11yOrderService.java       # Per-IndexView: position/key/focusType management
│   │       └── linking/
│   │           ├── A11yOrderLinking.java   # Singleton: orderKey → A11yLinkingQueue registry
│   │           ├── A11yLinkingQueue.java   # Per-group: sorted view chain via TalkBack APIs
│   │           └── WeakTreeMap.java        # Sorted weak-ref map (position → View)
│   ├── utils/
│   │   ├── A11yHelper.java                 # Static: a11y checks, DFS find, focus/keyboard helpers
│   │   ├── ChoreographerUtils.java         # Frame-based task scheduling (2-frame delay)
│   │   └── FragmentUtils.java             # Fragment lifecycle observer utilities
│   └── views/
│       ├── A11yCardView/                   # A11y.Card — card with accessible inner elements (TalkBack-compatible)
│       ├── A11yIndexView/                  # A11y.Index — positioned item in an order
│       ├── A11yOrderView/                  # A11y.Order — registers as order group container (extends ReactViewGroup directly)
│       ├── A11yLockView/                   # A11y.FocusTrap — traps TalkBack focus (modal pattern); includes A11yLockService
│       └── A11yPaneTitle/                  # A11y.PaneTitle — pane/screen transition announcements
├── newarch/                                # Fabric/TurboModule spec wrappers (Codegen): 6 files
└── oldarch/                                # Bridge spec wrappers: 6 files
```

## Core Protocols / Interfaces

| Interface | Defined in | Purpose |
|---|---|---|
| `A11yFocusProtocol` | services/focus/ | Contract for focusable views: `isViewFocused()`, extends `ViewParent`/`ViewManager` |

## Singleton Services

### A11yOrderLinking
Global registry mapping `orderKey` (String) → `A11yLinkingQueue`. The main coordination point between `A11yOrderView` (container) and `A11yIndexView` (items). Synchronized singleton; queues are auto-pruned when empty.

### A11yLinkingQueue
Owns a `WeakTreeMap` for one order group. When the map changes it links views via:
- `View.setNextFocusForwardId()` — keyboard navigation order
- `View.setAccessibilityTraversalBefore()` (API 22+) — TalkBack traversal order

### WeakTreeMap
`TreeMap<Integer, WeakReference<View>>`. Sorted by position. Dead references are auto-purged on `remove()`. Provides `getNext()` / `getPrev()` that skip GC'd entries, `forEachLive()` for iteration over live entries. Binary search via `NavigableMap`.

### A11yFocusService
Manages focus requests with retry logic. Uses `View.postDelayed()` for 300ms retries (max 3). Uses a `volatile boolean lock` to cancel pending retries once focus is acquired. Delegates actual focus posting to `ChoreographerUtils` to ensure layout stability. Also stores a weak reference to the last focused view via `storeViewReference()` / `getStoredView()`.

### A11yLockService
Stores weak references to the modal trap view and keyboard-focusable view. Used by `A11yLockView` to redirect out-of-bounds focus attempts.

## Focus Type (`orderFocusType`) in A11yOrderService

| Value | Constant | What gets linked |
|---|---|---|
| 0 | `ORDER_FOCUS_TYPE_DEFAULT` | The `A11yIndexView` itself |
| 1 | `ORDER_FOCUS_TYPE_CHILD` | First accessible descendant (DFS) |
| 2 | `ORDER_FOCUS_TYPE_LEGACY` | Stored child view or first subview |

## Dual Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArturKalach/react-native-a11y-order](https://github.com/ArturKalach/react-native-a11y-order) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
