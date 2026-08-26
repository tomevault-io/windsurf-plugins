---
trigger: always_on
description: This file applies to the entire repository. Read it before changing rendering,
---

# Agent Guide

This file applies to the entire repository. Read it before changing rendering,
the shared engine, native data transport, gestures, or performance-sensitive
code.

## Project purpose

`react-native-trading-charts` is a React Native New Architecture library for
native OHLC charts. React owns configuration and sends market data or commands;
native code owns chart state, gestures, snapshot construction, and rendering.
The library does not own networking.

The central design rule is: keep chart semantics and geometry in the shared C++
engine, and keep only platform rendering, lifecycle, gesture adapters, native
formatting, and React Native integration in the iOS and Android layers.

## Architecture map

- `src/TradingChartsView.native.tsx`: validates `chartId`, resolves config, and
  passes a memoized JSON config to the generated Fabric component. Unwraps all
  native events into plain payloads for public handlers.
- `src/TradingChartsViewNativeComponent.ts`: Fabric view codegen contract and
  the single source of truth for event payload types.
- `src/NativeTradingCharts.ts`: TurboModule command contract. The module is
  resolved lazily so importing the package never throws on non-native
  platforms.
- `src/TradingCharts.ts`: public imperative API. `addSeries` options are
  resolved/normalized in JS before crossing the bridge.
- `src/tradeBatcher.ts`: `createTradeBatcher`, the recommended way to feed
  high-frequency trade streams (one `updateTrades` call per interval).
- `cpp/chart_engine.{h,cc}`: shared candle store, trade aggregation, viewport,
  autoscale, ticks, crosshair selection, and snapshot publication.
- `cpp/internal/pane_layout.{h,cc}`: single source of truth for pane rects,
  shared by gesture hit-testing and the snapshot builder.
- `cpp/internal/render_snapshot_builder.{h,cc}`: immutable snapshot
  construction, visible-range computation, autoscale, ticks, and geometry
  tessellation (split into content and overlay vertex buffers).
- `cpp/internal/series_geometry.{h,cc}`: shared, allocation-free geometry
  strategies for candlestick, bar, and future price-series render types.
- `ios/TradingChartsView.mm`: Fabric view, gestures, on-demand frame scheduling,
  Metal renderer, and Core Animation text overlay.
- `ios/TradingChartsRegistry.mm`: main-thread command routing and bounded
  pending-command replay for views identified by `chartId`.
- `android/src/main/cpp/chart_engine_jni.cc`: JNI bridge to the shared engine and
  snapshot serialization.
- `android/src/main/java/com/tradingcharts/TradingChartsView.kt`: native view,
  gestures, frame coalescing, and lifecycle.
- `android/src/main/java/com/tradingcharts/ChartRenderer.kt`: GLES3 plot renderer.
- `android/src/main/java/com/tradingcharts/ChartOverlayView.kt`: Canvas text,
  badges, and tooltip overlay.
- `android/src/main/java/com/tradingcharts/TradingChartsRegistry.kt`: Android
  equivalent of the iOS registry.
- `cpp/tests/chart_engine_test.cc`: platform-independent behavior tests.

## End-to-end data and frame flow

1. JavaScript calls the TurboModule using a stable, unique `chartId`.
2. The platform registry dispatches on the main thread. If the Fabric view is
   not mounted, it retains commands and replays them when the view registers.
   A queued `setHistory` replaces older queued work. Pending queues are
   bounded; the oldest streaming commands (candle/trade batches) are dropped
   first so an unmounted view cannot grow memory without limit.
3. The native view mutates its `ChartEngine`; the engine increments `revision_`
   and marks its cached snapshot dirty.
4. The native view requests one frame. Repeated requests before the next vsync
   are coalesced. Data and gesture handlers request frames only when the
   engine reports an actual state change.
5. `ChartEngine::snapshot()` builds an immutable `RenderSnapshot` only when
   dirty; otherwise it returns the cached `shared_ptr`. When only crosshair
   state changed (`content_revision` unchanged), the builder copies the
   previous snapshot, shares its content vertex buffer without copying, and
   rebuilds only the crosshair overlay.
6. The GPU renderer consumes `content_vertices` and `overlay_vertices` from
   the same snapshot in two draws; the native overlay consumes ticks and
   metadata. `content_vertices` is re-uploaded only when `content_revision`
   changes; `overlay_vertices` follows every revision.
7. Visible-range events are emitted only when the first index, last index, or
   total candle count changes, and only while `has_visible_candles` is true.
   Preserve this guard to avoid unnecessary JS traffic.

## Shared C++ engine invariants

- Candle timestamps are milliseconds and must be non-decreasing. Empty time
  buckets are not synthesized. Older trades are ignored.
- `ChartEngine` is mutex-protected. Do not expose mutable candle or snapshot
  storage outside the lock.
- `RenderSnapshot` is immutable after publication. Preserve its lifetime until
  both the GPU and overlay have finished consuming it.
- `revision` identifies all render-relevant state. Any state mutation that
  changes output must call `markDirtyLocked()`.
- Geometry is interleaved as six floats per vertex: `x, y, r, g, b, a`, and is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kirill3333/react-native-trading-charts](https://github.com/kirill3333/react-native-trading-charts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
