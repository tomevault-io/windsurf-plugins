---
trigger: always_on
description: Guidance for coding agents working on Actualist.
---

# AGENTS.md

Guidance for coding agents working on Actualist.

## Project Intent

Actualist is a native iOS 26+ local-first client for Actual Budget. It talks to the normal Actual server sync API, stores an imported SQLite budget locally, applies CRDT messages, and renders from the local database. The app no longer relies on the sibling `actual-http-api` package or its REST/OpenAPI contract. Preserve the visual direction: dark, compact, rounded, money-forward, Liquid Glass-aware, and optimized for repeated budget review.

## Current References

- Development pipeline: `docs/DEVELOPMENT.md`
- Mechanical gate: `scripts/check.sh`. Run it before handing off a change. It
  covers whitespace, Liquid Glass, TestFlight notes, synchronized-group
  integrity, and file-size signals. It does not replace tests.
- Simulator and device ids for this machine: gitignored
  `scripts/lib/destinations.sh` (copy `scripts/lib/destinations.example.sh`).
  Pin destinations by UDID, never by display name.

## Local-First Backend

- The app connects to a normal Actual server, not the `actual-http-api` REST wrapper.
- Sync traffic uses Actual's `/sync/sync` protocol and local CRDT message storage.
- Reads must come from `LocalFirstActualStore` and `BudgetDatabase`, not from HTTP REST endpoints.
- Writes must generate Actual-compatible CRDT messages, apply them to SQLite, enqueue them in `actualist_outbox`, then reload local caches and opportunistically flush.
- Do not commit sync tokens, passwords, encryption keys, budget IDs, imported databases, or personal financial data.

## Development Defaults

- Use Swift and SwiftUI only for app UI.
- Do not introduce UIKit UI code.
- Use a standard Xcode SwiftUI app project unless the user explicitly changes direction.
- Target iOS 26+.
- Enforce iOS 26 Liquid Glass for all glass-like controls, buttons, toolbars, floating navigation, and panels. Use only public SwiftUI Liquid Glass APIs:
  - `.buttonStyle(.glass)`
  - `.buttonStyle(.glassProminent)`
  - `.buttonStyle(.glass(...))`
  - `.glassEffect(_:in:)`
- Liquid Glass must be system-owned wherever SwiftUI provides native chrome. Do not wrap native toolbar buttons, tab bars, navigation bars, sheets, alerts, or menus in custom glass containers.
- Do not apply `.buttonStyle(.glass)`, `.buttonStyle(.glassProminent)`, or `.buttonStyle(.glass(...))` to buttons inside a SwiftUI `.toolbar`. Let the toolbar render its own Liquid Glass button chrome. Toolbar labels should usually be plain `Button` views with SF Symbols and optional font/control-size adjustments only.
- The main app navigation must use native `TabView` with `.tabItem`. Do not recreate the tab bar with custom `HStack`, `ZStack`, `safeAreaInset`, overlay, capsule, or `FloatingTabBar` views.
- Never place a glass-styled button inside a view that already has `.glassEffect`, and never place a `.glassEffect` wrapper around a native glass button. This creates the visible "button inside button" defect on device.
- Use `.glassEffect(_:in:)` only for standalone non-control panels or custom surfaces that are not themselves native SwiftUI chrome. If the element is clickable and should look like a button, prefer the appropriate native `Button` style rather than wrapping it in another glass shape.
- Do not use `GlassEffectContainer` in this app until it has been explicitly re-tested on a physical iOS 26 device. The first physical-device run after adding it crashed before app code with a system `OS_dispatch_mach_msg _setContext:` selector failure.
- Do not fake Liquid Glass with `.regularMaterial`, `.thinMaterial`, `.ultraThinMaterial`, `.thickMaterial`, blur overlays, translucent hand-rolled capsules, or custom material-backed toolbar containers.
- Row hit areas may still use `.buttonStyle(.plain)` when they should look like list rows instead of controls.

### Liquid Glass Examples

Toolbar buttons must be plain toolbar content. The toolbar supplies the glass.

Wrong:

```swift
ToolbarItem(placement: .topBarTrailing) {
    Button {
        Task { await load() }
    } label: {
        Image(systemName: "arrow.clockwise")
    }
    .buttonStyle(.glass(.clear))
    .glassEffect(.regular, in: Circle())
}
```

Right:

```swift
ToolbarItem(placement: .topBarTrailing) {
    Button {
        Task { await load() }
    } label: {
        Image(systemName: "arrow.clockwise")
    }
    .font(.body.weight(.semibold))
    .controlSize(.small)
}
```

The main app tab bar must be native `TabView`, not a custom floating glass control.

Wrong:

```swift
ZStack(alignment: .bottom) {
    content

    HStack {
        Button("Budget") { selectedTab = .budget }
            .buttonStyle(.glass(.regular))
        Button("Accounts") { selectedTab = .accounts }
            .buttonStyle(.glass(.regular))
    }
    .glassEffect(.regular, in: Capsule())
}
```

Right:

```swift
TabView(selection: $selectedTab) {
    BudgetView()
        .tabItem {
            Label("Budget", systemImage: "list.bullet.rectangle.portrait.fill")
        }
        .tag(AppTab.budget)

    AccountsView()
        .tabItem {
            Label("Accounts", systemImage: "building.columns.fill")
        }
        .tag(AppTab.accounts)
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sporez/actualist](https://github.com/sporez/actualist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
