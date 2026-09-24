---
trigger: always_on
description: Goal: shortest path from user request to the exact files, commands, and constraints needed to do the work.
---

# Codex Instructions

Goal: shortest path from user request to the exact files, commands, and constraints needed to do the work.

## Start Here

| Task | Open First | Usually Validate With |
|---|---|---|
| BLE protocol, BT probe, BT device bug | `docs/protocol/PROTOCOL.md` then `docs/protocol/BLE_PROTOCOL.md`; `OpenSnek/Sources/OpenSnekProtocols/BLEVendorProtocol.swift`; `OpenSnek/Sources/OpenSnekHardware/BLEVendorTransportClient.swift`; `OpenSnek/Sources/OpenSnek/Bridge/BridgeClient+Bluetooth.swift`; `OpenSnek/Sources/OpenSnekProbe/{main.swift,ProbeTransport.swift}` | `swift test --package-path OpenSnek --filter BLEVendorProtocolTests`; build/run probe |
| Windows Synapse/BTVS capture, profile reverse engineering | `captures/README.md`; `docs/protocol/BLE_PROFILE_CRUD_SPEC.md`; `docs/research/BASILISK_V3_PRO_BT_EXTENDED.md`; run `tools/windows/capture-btvs.ps1` and inspect `synapse-events.md`, `correlation.md`, then `summary.md` | focused BTVS capture plus Synapse log correlation artifacts |
| USB protocol, USB lighting, USB buttons | `docs/protocol/PROTOCOL.md` then `docs/protocol/USB_PROTOCOL.md`; `OpenSnek/Sources/OpenSnekProtocols/USBHIDProtocol.swift`; `OpenSnek/Sources/OpenSnek/Bridge/BridgeClient+USB.swift`; `OpenSnek/Sources/OpenSnekCore/DeviceSupport.swift` | focused USB probe command; `DeviceProfilesTests`; `USBButtonHydrationTests` |
| Device support, product IDs, zones, button layout | `OpenSnek/Sources/OpenSnekCore/{DeviceSupport.swift,Models.swift,ButtonBindingSupport.swift}`; `docs/protocol/PARITY.md` if shipped-status changes | `swift test --package-path OpenSnek --filter DeviceProfilesTests` |
| App-state hydration, persistence, auto-apply | `OpenSnek/Sources/OpenSnek/Services/{AppState.swift,AppStateEditorController.swift,AppStateApplyController.swift,DeviceStore.swift,EditorStore.swift}`; `OpenSnek/Sources/OpenSnekAppSupport/DevicePreferenceStore.swift` | `swift test --package-path OpenSnek --filter AppStateRefactorCharacterizationTests` |
| Background service, bridge transport, snapshots | `OpenSnek/Sources/OpenSnek/Services/{BackendSession.swift,BackgroundServiceCoordinator.swift,AppStateRuntimeController.swift}`; `OpenSnek/Sources/OpenSnek/Bridge/BridgeClient.swift` | `swift test --package-path OpenSnek --filter BackgroundServiceTransportTests` or `RemoteServiceSnapshotTests` |
| UI, menu bar, startup/lifecycle | `OpenSnek/Sources/OpenSnek/UI/*.swift`; `OpenSnek/Sources/OpenSnek/{AppLifecycleDelegate.swift,OpenSnekApp.swift}`; `RuntimeStore.swift` | `swift test --package-path OpenSnek --filter AppLifecycleDelegateTests` or `ServiceMenuBarPresentationTests` |

Protocol behavior changes require docs, tests, and `CHANGELOG.md` updates in the same change.

## Canonical Sources

- Swift app/probe code and protocol docs are canonical.
- Python tooling (`tools/python/`) is useful for probing and comparison, but may lag; do not treat it as source of truth when it disagrees with Swift/docs.
- Open `docs/protocol/PARITY.md` only when support status, shipped capability, or transport parity changes.

## Current Validated Devices

- Basilisk V3 X HyperSpeed: USB `0x00B9`, Bluetooth `0x00BA`
- Basilisk V3 Pro: USB `0x00AB`, Bluetooth `0x00AC`
- Basilisk V3 35K: USB `0x00CB`

## Repo Rules

1. BLE vendor exchanges stay serialized one-at-a-time per connection.
2. Command-level retries are forbidden. Retries may only live in protocol/transport connection recovery for real connection errors; command failures must surface clearly instead of being papered over by repeating the command while device state settles.
3. When the user says something is broken, assume it is a regression unless evidence says otherwise. Do not guess or stack speculative fixes. Start from logs and existing diagnostics; if the available data is insufficient, add targeted logging/diagnostics, walk the user through a repro, then author the fix from the captured repro evidence.
4. For connection-state regressions, especially USB/Bluetooth presence, reachability, stale HID sessions, reconnect settle timing, background-service snapshots, and UI availability, treat disconnect and reconnect as one behavioral surface. Before changing that surface, inspect `git blame`, commit history, and relevant PR history for the touched paths; identify the previously working behavior; then add or update regression coverage for both directions: unplug/offline/sleep and replug/wake/recover. Do not ship a one-sided fix that proves only disconnect or only reconnect. If hardware validation is required and unavailable, add targeted diagnostics or a manual repro checklist and state the gap clearly.
5. Prefer focused reads, focused tests, and the smallest useful probe/build command instead of defaulting to full-package runs.
6. Keep latest-wins/coalesced apply behavior for rapid UI edits.
7. Treat malformed BLE DPI payloads as transient; ignore them instead of applying bad state or retrying the command.
8. For BLE DPI stages, preserve stage IDs on write, resolve active stage from stage IDs, and do not reintroduce stage nudge/toggle writes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gh123man/OpenSnek](https://github.com/gh123man/OpenSnek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
