---
trigger: always_on
description: Native macOS app for Android/React-Native debugging over adb. A Raycast-style
---

# CLAUDE.md — Droidective

Native macOS app for Android/React-Native debugging over adb. A Raycast-style
command palette: searchable feature sidebar, persistent device bar, a detail
pane per feature. Swift 6 + SwiftUI, macOS 14+.

## Architecture (the load-bearing rule)

Two layers, strictly separated so a second **Apple** UI (iPad/visionOS) could
reuse ADBKit almost as-is. The Windows/Linux port is now scheduled and staged
(strategy + phases in `docs/cross-platform.md`): ADBKit compiles and runs its
whole suite on Linux (CI `test-linux`; `make test-linux` locally) and on
Windows (CI `build-windows`). The Apple-bound subsystems — the Mirror media
stack, the Network.framework servers (Reactotron, the JS-console test fake),
`NSDataDetector`, `proc_pid_rusage` — are `#if canImport`-gated out rather than
stubbed; the portable seams (`HostArchive` extraction, `FileHandleLines`,
per-OS `ToolLocator`, swift-crypto digests off-Apple) carry everything else.
Phase 2 landed `droidectived/`, a local daemon over ADBKit
(`docs/droidectived-protocol.md`); phase 3 is `desktop/`, the Tauri 2 + React
UI over it for Windows/Linux (`desktop/README.md`; the feature-by-feature
parity tracker is `docs/desktop-parity.md`). **macOS never talks to the
daemon** — the Mac app keeps linking ADBKit directly, by decision, so no daemon
or desktop work can reach the shipping Mac flow.

**The desktop UI is the Mac's UI.** The Mac app is the proven one; the point of
the port is that someone moving between the two does not have to relearn
anything. So where a control exists on both, it looks and behaves the way
`App/Sources/` makes it behave — same wording, same icon, same confirmation
shape, same gesture (a double-click to open stays a double-click; a
`confirmationDialog` stays a dialog and does not become an armed button). A
nicer idea for Windows and Linux is still a difference someone has to relearn:
if it is genuinely better it goes into the Mac app *first*, and the port
follows. Two standing exceptions, and they are named where they occur: a
keyboard shortcut whose modifier has no Windows/Linux equivalent (⌘\ became
Ctrl+\, and the split is Ctrl+\ not Ctrl+D because Ctrl+D is end-of-input in
every Linux shell), and a label that names a platform. **Every** feature is in
scope, chrome included — Settings, the notification panel, toasts, the role
picker, the catalog, the menu bar, drag and drop. Only `ios-logs` and
`push-notification` are out, and only because `xcrun simctl` is a macOS
toolchain rather than anything about a device. An iOS companion can't run
`Process` at all, so it would ride the same daemon protocol. Keep the seams (`ProcessRunning`, injected
directories) intact, and keep new ADBKit code portable — no new Apple-only
framework use outside the gated subsystems. The rule is **enforced, not just
documented**: `PortabilityGuardTests` scans ADBKit and fails on an Apple-only
import or a corelibs trap that isn't inside a matching `#if canImport(...)`
gate. Its allowlist is empty, and a companion test fails on a stale entry.

- **`ADBKit/`** — a SwiftPM package holding *all* logic. Zero UI imports
  (feature icons are SF Symbol *name strings*). Actors for stateful services,
  `Sendable` value types, strict concurrency complete. Test with
  `cd ADBKit && swift test` — no Xcode, no device needed.
- **`App/`** — thin SwiftUI shell, split in two `@Observable @MainActor` halves:
  **`AppCore`** is the app (one `adb devices` poll, tool caches, the persisted
  feature curation, the Reactotron/MCP listeners, the window registry), and
  **`AppState`** is *one window's workspace* (device, tabs, terminals, JS
  console). The app is multi-window — one window per device — and `AppState`
  forwards everything app-wide, so a feature view reads `state.devices` /
  `state.layout` without knowing which window it's in. See the multi-window
  convention below and `docs/multi-window.md`. Built via XcodeGen
  (`project.yml`) + xcodebuild; `.xcodeproj` is gitignored and regenerated.

When adding a feature: logic + a parser test go in ADBKit; the view goes in
`App/Sources/FeatureDetail/Views/`. Never put adb/Process logic in a SwiftUI view.
Follow the **Adding a feature** checklist below — a feature's `id` is a contract
spread across several files, and most omissions fail *silently* (a "Coming Soon"
screen), not loudly.

## Adding a feature — the checklist

A feature's string `id` is the contract across several files. Do these in order.
**[test]** steps fail `swift test` — or the AppTests bundle, which CI runs —
if skipped; **[silent]** steps have *no*
automated guard, so the failure mode is a non-working feature you only catch by
opening it — verify those by hand.

1. **Define it** — add a `FeatureDef` to `FeatureRegistry.all` (unique `id`,
   title, keywords, category, `kind`; set `platforms` if it works on iOS
   Simulators — the default is Android-only). **[test: `hasAll60Features` —
   bump the count; `byID` traps on a duplicate id]**
2. **If it's an action** (`.instantAction`/`.formAction`/`.toggleAction`):
   - add the runner `case` to `FeatureEngine.dispatch`,
   - add the `id` to `FeatureEngine.implementedIDs`,
   - add an arg-vector test in `FeatureEngineTests` asserting the exact adb

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Droidective/Droidective](https://github.com/Droidective/Droidective) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
