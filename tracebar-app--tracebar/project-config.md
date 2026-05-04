---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TraceBar is a macOS menubar app providing continuous graphical traceroute monitoring (like `mtr`). Built with Swift + SwiftUI, targeting macOS 14.6+. Bundle ID: `org.evilscheme.TraceBar`, Dev Team: `4PX677GC4R`.

## Development Guidelines
- whenever thinking about color, make sure colors are selected that work with the color theme system (ColorTheme.swift)
- use the xcode MCP server if configured when interacting with xcode
- use the xcode MCP and/or the context7 MCP to look up swift/MacOS API details

## Architecture

Single-process sandboxed app using unprivileged ICMP sockets (`SOCK_DGRAM`).

```
TraceBar/TraceBar/
  TraceBarApp.swift              — @main, MenuBarExtra + Settings scene
  ViewModels/
    TracerouteViewModel.swift    — @MainActor ObservableObject, all app state + probe scheduling
  Views/
    MenuBarView.swift            — MenuBarView: renders NSImage for menubar (chart + latency text)
    SparklineChart.swift         — Canvas line chart renderer
    HeatmapChart.swift           — Canvas heatmap grid renderer
    VerticalBarsChart.swift      — Canvas vertical bars renderer
    BandwidthChart.swift         — Dual-axis bandwidth bars
    DetailViewPanel.swift        — Main dropdown panel
    HopRowView.swift             — Single hop row with stats + history chart
    ChartTooltip.swift           — Tooltip system for mouse-hover on charts
    ChartMode.swift              — Enum defining the three chart modes
    SettingsView.swift           — General + Appearance + Advanced tabs
  Models/
    TracerouteModels.swift       — ProbeResult, HopData (stats computed from RingBuffer)
    ColorTheme.swift     — 15 color schemes with gradient interpolation
    BandwidthModels.swift        — BandwidthSample
    RingBuffer.swift             — Generic circular buffer
  Services/
    ICMPEngine.swift             — SOCK_DGRAM ICMP sockets, probeRound() (serial queue only)
    BandwidthMonitor.swift       — Interface detection via routing socket, sysctl byte counters
    WindowManager.swift          — @MainActor singleton, observes window notifications, ensures settings/panel ordering
tools/
  mtr-lite.swift                 — CLI traceroute tool (compiles with ICMPEngine.swift)
```

### Concurrency Model
- **@MainActor:** TracerouteViewModel — all UI state, @Published properties
- **Serial probe queue (GCD):** ICMPEngine + BandwidthMonitor must be called from single serial DispatchQueue
- **Receiver thread:** Global concurrent queue with blocking recvfrom() in probeRound()
- **Timing:** mach_absolute_time() for sub-ms latency measurement
- **Debouncing:** DispatchWorkItem for settings-triggered reschedule

### Key Types
- `ProbeResult` — single probe: hop, address, latency, timestamp
- `HopData` — accumulated hop: address, hostname, RingBuffer of probes, computed last/avg/loss
- `RingBuffer<T>` — generic circular buffer, chronological iteration
- `ColorTheme` — enum with 15 schemes, 2-3 stop RGB gradient interpolation
- `BandwidthSample` — timestamp, download/upload bytes per sec, interface name

### Settings (@AppStorage keys)
`targetHost` (8.8.8.8), `resolveHostnames` (true), `colorScheme` (thermal), `showBandwidth` (true), `showMenuBarBackground` (true), `compactMenubar` (false), `menubarChartMode` (sparkline), `chartMode` (heatmap), `idleProbeInterval` (10s), `activeProbeInterval` (1s), `historyMinutes` (3), `maxHops` (30), `latencyThreshold` (100ms). Launch at login via SMAppService.

### Dependencies
- **swift-snapshot-testing** (Point-Free, v1.19.1) — test-only, image snapshot comparison
- Foundation, SwiftUI, Darwin (sockets, mach timing), AppKit (NSImage/NSColor for menubar), ServiceManagement

**Entitlements:** `com.apple.security.app-sandbox`, `com.apple.security.network.client`, `com.apple.security.network.server`. The `SOCK_DGRAM` + `IPPROTO_ICMP` approach requires no root privileges and works inside App Sandbox.

## Build & Run

Open `TraceBar/TraceBar.xcodeproj` in Xcode. Single target: TraceBar.

## Tests

Uses Swift Testing framework (`@Test`, `@Suite`) for unit tests and XCTest + swift-snapshot-testing for snapshot tests. Run via Xcode (Cmd+U) or:
```bash
xcodebuild test -project TraceBar/TraceBar.xcodeproj -scheme TraceBar -destination 'platform=macOS'
```

### Unit Tests (Swift Testing)
| File | Coverage |
|------|----------|
| RingBufferTests.swift | empty, append, wraparound, capacity, init from array |
| TracerouteModelsTests.swift | ProbeResult timeouts, HopData stats |
| ICMPParsingTests.swift | Echo Reply, Time Exceeded, Dest Unreachable, identifier validation |
| ColorThemeTests.swift | boundary colors, interpolation, clamping |

### Window Behavior Tests (Swift Testing)
| File | Coverage |
|------|----------|
| WindowManagerTests.swift | register, reset, ensureSettingsVisible, reorderWindows |
| SettingsWindowTests.swift | deactivation survival, level ordering, panel coexistence |
| TooltipWindowTests.swift | child window attach, hide, cleanup |

### Snapshot Tests (XCTest + swift-snapshot-testing)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tracebar-app/tracebar](https://github.com/tracebar-app/tracebar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
