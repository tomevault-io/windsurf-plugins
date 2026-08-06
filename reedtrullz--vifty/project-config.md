---
trigger: always_on
description: AI coding instructions for working in this repository.
---

# AGENTS.md — Vifty

AI coding instructions for working in this repository.

## Build System

- Swift Package Manager (`Package.swift`, tools-version 6.0).
- `swift build` / `swift test` / `make test-fast` / `make test-full` / `make verify` / `make verify-full` / `make app` / `make validation-evidence` (see Makefile).
- macOS 15 minimum deployment target.
- `.build/` is gitignored.

## Target Layout

| Target | Type | Dependencies |
|--------|------|-------------|
| Vifty | executable | ViftyCore |
| ViftyCore | library | ViftyPrivateIOKit |
| ViftyDaemon | executable | ViftyCore |
| ViftyHelper | executable | ViftyCore |
| ViftyCtl | executable | ViftyCore |
| ViftyPrivateIOKit | C target | IOKit framework |
| ViftyCoreTests | test | ViftyCore, Vifty |

ViftyCore links `IOKit.framework` and ViftyPrivateIOKit links it too (C target needs explicit linking).

## Key Files

- `Sources/ViftyCore/Models.swift` — All data types: Fan, TemperatureSensor, HardwareSnapshot, FanCurve, CurveProfile, FanMode, FanCommand, ControlState, ViftyError.
- `Sources/ViftyCore/AgentControlModels.swift` — Codable agent-control requests, leases, policy snapshots, decisions, retry metadata, and status.
- `Sources/ViftyCore/AgentControlPolicy.swift` — conservative policy for bounded workload leases.
- `Sources/ViftyCore/AgentControlService.swift` — daemon-owned service that applies agent cooling targets and restores Auto.
- `Sources/ViftyCore/HardwareService.swift` — `HardwareService` protocol + `FanControlCoordinator` actor + `ManualControlMarker`.
- `Sources/ViftyCore/RealMacHardwareService.swift` — `RealMacHardwareService` (daemon-first SMC reads/writes, local fallback).
- `Sources/ViftyCore/CurveProfileStore.swift` — JSON file persistence for saved curve profiles.
- `Sources/ViftyCore/SMCClient.swift` — IOKit SMC connection, read/write allowlist, SMCValue, SMCDecoding (float, FPE2, flt, uint en/decoding).
- `Sources/ViftyCore/FanInfoReader.swift` — Pure SMC fan snapshot parser for hardware Auto/Forced/System mode, mode-key casing probes, and target RPM.
- `Sources/ViftyCore/FanDisplayFormatter.swift` — Pure fan-state display strings for UI rows.
- `Sources/ViftyCore/HardwareSnapshotProbeFormatter.swift` — Pure helper-probe text formatter for hardware validation evidence.
- `Sources/ViftyFanControlSafety/LocalFanHelperClient.swift` — Internal transactional SMC fan writer with preflight, readback, rollback, mode-key probing, and guarded `Ftst` handling.
- `Sources/ViftyCore/PowerInfo.swift` — Local IOKit power telemetry parser (`IOPS`, `AppleSmartBattery`, adapter details) + UI formatters.
- `Sources/ViftyCore/ThermalPressure.swift` — macOS thermal-pressure state model and display helpers.
- `Sources/ViftyCore/TelemetryHistory.swift` — In-memory rolling telemetry sample buffer.
- `Sources/ViftyCore/ViftyCtlArguments.swift` — pure parser for the bundled agent CLI, including read-only audit export options.
- `Sources/ViftyCore/ViftyCtlReadinessReport.swift` — machine-readable `viftyctl diagnose` report for hardware/agent readiness.
- `Sources/ViftyCore/ViftyCtlRunner.swift` — testable command runner used by `viftyctl`, including structured capabilities, read-only audit export, and retry handling.
- `Sources/ViftyCore/XPCAuditTokenCoding.swift` — audit-token byte bridge used by the daemon XPC identity extractor.
- `Sources/ViftyCore/ViftyDaemonClient.swift` — XPC client that talks to the privileged daemon.
- `Sources/ViftyCore/ViftyDaemonProtocol.swift` — `@objc` XPC protocol + `XPCSnapshotCoding` / `XPCAgentControlCoding` bridges for snapshots, agent status, leases, and audit events.
- `Sources/ViftyDaemon/main.swift` — XPC listener with `DaemonService` exporting the protocol.
- `Sources/ViftyHelper/main.swift` — CLI for `probe`, `readKey`, `setFixed`, `auto`, `smcDiagnostics`.
- `Sources/ViftyCtl/main.swift` — thin `viftyctl` command entrypoint.
- `Sources/Vifty/ViftyApp.swift` — `@main` SwiftUI app entry (menu bar extra + window scene).
- `Sources/Vifty/AppModel.swift` — `@MainActor ObservableObject` driving UI polling, fan/profile state, and power snapshot refresh.
- `Sources/Vifty/AppPreferencesStore.swift` — Codable private JSON persistence for startup/default mode, menu-bar display/custom fields, Codex display settings, fixed per-fan targets, and local notification settings, with legacy UserDefaults migration.
- `Sources/Vifty/SoftwareUpdate.swift` — Developer-ID-gated advisory GitHub release checker, strict expected-asset metadata validation, single-owner private update preference/cache state, scheduling, and fixed release-page browser handoff; it never downloads or installs executable code.
- `Sources/Vifty/CodexUsage.swift` — optional local Codex app-server/session-log usage reader plus menu-bar display formatting for text and battery-style usage modes.
- `Sources/Vifty/LocalNotifications.swift` — opt-in local UserNotifications wrapper and notification settings model for helper, thermal-pressure, restore-failure, plugged-in-drain, and agent-cooling attention alerts.
- `.github/workflows/ci.yml` — GitHub Actions CI: Swift tests, release app build, plist/code-sign checks, temp install verification, and app artifact upload.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Reedtrullz/Vifty](https://github.com/Reedtrullz/Vifty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
