---
trigger: always_on
description: Sideload-only iPhone battery instrument. Private APIs; never App Store safe.
---

# MiniWatts

Sideload-only iPhone battery instrument. Private APIs; never App Store safe.

The method for reading the PMU — dlsym'd IOKit, `IOHIDEventSystemClient` on usage
pages `0xff08` and `0xff00` — and the first version of the *Verified blocked* list
below are taken from
[ios-charging-monitor](https://github.com/gregsramblings/ios-charging-monitor)
(ChargeSpeed, MIT). What is built on that is this project's own: the five screens,
the thermal zone mapping, the USB-PD inspector, the energy integration and session
history, and everything under *Sensor notes*, which the blocked list has since grown
by several entries.

## Build and run

- Plain Xcode project, no XcodeGen, no packages. `PBXFileSystemSynchronizedRootGroup`:
  new files under `MiniWatts/` join the target automatically, do not edit the pbxproj.
- iOS 17 deployment target, **Swift 6** language mode, Xcode 26+ required
  (`nonisolated` on type and extension declarations is Swift 6.2).
- `./scripts/build-ipa.sh` — unsigned ipa, the distributable one; runs
  `verify-clean.sh` on itself and fails if the artifact carries identifying data.
- `TEAM_ID=… ./scripts/build-ipa.sh signed` — for your own device. No default team
  lives in this repo; `DEVELOPMENT_TEAM` is empty in the pbxproj and Xcode will
  write yours back into it if you pick one in the UI. Do not commit that.
- `.github/workflows/build.yml` builds, verifies and (on a `v*` tag) releases.
- The simulator reads the **Mac's** battery through IOKit and has no HID sensors:
  fine for layout and for the adapter/PD panels, useless for anything sensor-driven.

## Layout

- `Core/Sensors/` — the probes. `IOKitBattery` (dlsym'd IOKit + powerd), `HIDSensors`
  (`IOHIDEventSystemClient`, one client per process, created once), `BatteryCenterBridge`,
  `ThermalMonitor` (`ProcessInfo.thermalState`, public API), `SensorCatalog`
  (name → zone/label by whole-word keyword, never exact).
- `Core/Model/` — `PowerSnapshot` merges all four sources and owns every derived value.
  Anything derived from the HID readings is resolved **once, in `init`, and stored** —
  it used to be computed per access, which meant a body asking for the battery
  temperature four times did four linear scans and called `SensorCatalog.zone(for:)`
  (which lowercases and splits) once per sensor per scan. Registry lookups stay
  computed: those are single hash hits. Also here:
  `EnergyAccumulator` integrates ∫V·I dt; `ChargeSession` + `SessionStore` persist charges
  as one JSON file in Application Support.
- `Core/PowerMonitor.swift` — `@Observable`, 1 s tick, drives everything and owns session
  lifecycle. Injected once in `MiniWattsApp`, read via `@Environment(PowerMonitor.self)`.
  `headline` lives here rather than on the snapshot: its last fallback is the %-rate
  estimate, which is derived across several snapshots and so is not a snapshot's to give.
- `Design/` — palette (`Color.mw(light:dark:)`, no asset catalog entries), `Panel`/
  `Metric`/`Pill`/`BarRow`, `PowerRing`, Swift Charts wrappers, `PhoneHeatMap`.
- `Features/` — one folder per tab, plus Settings. `DebugView` (Raw data) is
  `#if DEBUG` only and reached from the bottom of Settings, not the main toolbar.

## Swift 6 isolation

The project sets `SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor`, so everything is
main-actor isolated unless it says otherwise. Consequences that have already bitten:

- **`Theme.swift` must stay `nonisolated`.** `Color.mw` builds a `UIColor` with a
  trait-resolution closure, and UIKit calls that from whatever thread is resolving a
  dynamic colour while rendering. Under Swift 6 the compiler inserts an executor
  check there, and the app **traps on the first frame that paints a gradient**. It
  compiles fine either way — this is a runtime crash, not a build error.
- The whole `Core/` layer is `nonisolated`: it has no UI in it, and nonisolated
  protocol requirements (`Shape`, `Layout`, `Identifiable`, `Codable`) cannot be
  witnessed by main-actor-isolated members. `PowerMonitor` and `ThermalMonitor`
  stay on the main actor — they are `@Observable` UI state.
- `deinit` is nonisolated in Swift 6 and cannot touch isolated stored properties.
  `ThermalMonitor` therefore has no notification observers to tear down; it is
  polled from the one-second tick instead, which costs nothing and covers the same
  ground (nothing observes a change made while the app is suspended anyway).

## Sessions and the app lifecycle

A charge session ends when the **charger is unplugged**, not when the app leaves the
foreground. Three things make that work and they are easy to undo by accident:

- `RootView` calls `monitor.pause()` on `.background` only. It used to call a `stop()`
  that closed the session on anything that was not `.active`, and `.inactive` fires for
  a pulled-down Control Center, the app switcher, an incoming call and the screen
  locking — so an overnight charge was recorded as a scatter of two-minute fragments.
- `closeSessionIfNeeded` dates the end from `lastConnectedObservation`, the last tick
  that actually saw a charger, not from `.now`. Unplug while the app is suspended and
  the first tick after it wakes is the first that knows; `.now` there would stretch the
  session across however long the app was away.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ResistanceTo/MiniWatts](https://github.com/ResistanceTo/MiniWatts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
