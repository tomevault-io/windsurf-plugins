---
trigger: always_on
description: macOS menu bar app that monitors and blocks AWDL (Apple Wireless Direct Link) to reduce WiFi latency spikes during cloud gaming. Swift/SwiftUI, macOS 13+.
---

# Ping Warden

macOS menu bar app that monitors and blocks AWDL (Apple Wireless Direct Link) to reduce WiFi latency spikes during cloud gaming. Swift/SwiftUI, macOS 13+.

## Key Components

| Component | Purpose |
|-----------|---------|
| `PingWarden` (main app) | SwiftUI menu bar app, preferences, UI |
| `PingWardenHelper` | Privileged SMAppService daemon, manages AWDL via `ifconfig` |
| `PingWardenWidget` | macOS Control Center widget |
| XPC | App↔helper communication |

**Bundle IDs:** `com.amesvt.pingwarden` (app), `com.amesvt.pingwarden.widget`, `com.amesvt.pingwarden.helper`
**Team ID:** `PV3W52NDZ3`
**App Group:** `PV3W52NDZ3.com.amesvt.pingwarden` (Team-ID-prefixed so Developer ID builds work without an embedded provisioning profile)

## Source Structure

```
PingWarden/
├── PingWarden/               # Main app (Swift/SwiftUI)
│   ├── Core/                 # PingStatistics, TCPProbe, XPCReconnectPolicy
│   ├── PingWardenApp.swift   # Entry point, menu bar setup
│   ├── PingWardenMonitor.swift  # AWDL blocking logic, stateLock
│   ├── PingMonitor.swift     # Real-time TCP latency monitoring
│   ├── DashboardView.swift   # Live ping dashboard
│   ├── MonitoringStateStore.swift  # Persisted monitoring state
│   ├── DiagnosticsExporter.swift  # Diagnostic data export
│   ├── QuarantineHelper.swift     # macOS quarantine attribute handling
│   ├── ControlCenterSupport.swift # Control Center widget integration
│   ├── PingWardenPreferences.swift # UserDefaults wrapper
│   ├── notarize.sh           # Notarization + stapling
│   └── release.sh            # DMG creation, appcast signing, GitHub release
├── PingWardenHelper/         # Privileged daemon (Obj-C)
│   ├── main.m                # Daemon entry point
│   └── PingWardenMonitor.h/.m  # AF_ROUTE socket listener, ifconfig calls
├── PingWardenWidget/         # macOS Control Center widget (toggle intent + preferences)
├── Common/                   # Shared XPC protocol (HelperProtocol.h)
├── create-dmg.sh             # DMG packaging with background image
└── PingWarden.xcodeproj
```

## Build

```bash
cd PingWarden
xcodebuild -project PingWarden.xcodeproj -scheme PingWarden -configuration Release
```

## Testing

The Foundation-only core helpers are covered by a SwiftPM test target driven
by `Package.swift` at the repo root. The Xcode app build is unaffected — the
package's `PingWardenCore` target points at `PingWarden/PingWarden/Core/` via
an explicit `path:` so the same Swift sources back both build systems.
Core helpers and tests stay Foundation/POSIX-only so CI can run them on both
the macOS 26 runner and an `ubuntu-latest` Swift container.

```bash
swift test            # canonical command
./scripts/run-smoke-tests.sh   # thin wrapper, runs `swift test`
```

Coverage: `PingStatistics.calculate()` edge cases (empty, healthy, lossy,
even-count median, fair band, min/max, all-failures, single-sample jitter),
`XPCReconnectPolicy.delayForAttempt` (backoff curve, 30 s cap, monotonicity),
`TCPProbe` failure and success paths (invalid hostname, closed loopback port,
open loopback port), `StateObserverRegistry` add/remove/snapshot lifecycle,
`VersionPromptPolicy` and custom-target boundaries, and `HelperBundleValidator`
failure modes (missing binary, missing plist, non-executable binary, valid
bundle).

## Release Process

`notarize.sh` pre-validates notarytool credentials automatically (it calls
`xcrun notarytool history --keychain-profile` and aborts with a setup hint
before doing any work). If the profile ever expires, run:
```bash
xcrun notarytool store-credentials "notarytool-profile"
```

**Do NOT use `xcodebuild -exportArchive`** — broken in Xcode 26 (IDEDistributionMethodManagerErrorDomain Code=2). Archive **unsigned** and let `notarize.sh` apply the real Developer ID signature (fully headless, no Xcode UI needed):
1. `xcodebuild archive -project PingWarden.xcodeproj -scheme PingWarden -configuration Release -archivePath /tmp/PingWarden-X.Y.Z.xcarchive -destination 'generic/platform=macOS' CODE_SIGNING_ALLOWED=NO CODE_SIGNING_REQUIRED=NO`. The `CODE_SIGNING_ALLOWED=NO` is **required** so the archive is deterministic and does not depend on Xcode's provisioning UI. The app uses a Team-ID-prefixed macOS App Group that does not require a profile. `notarize.sh` re-signs every component with `codesign --entitlements` (Developer ID, no profile). The archive's signing is throwaway. The archive yields both the `.app` **and** `dSYMs/` (the latter is needed by `release.sh` for Sentry).
2. `rsync -a "/tmp/PingWarden-X.Y.Z.xcarchive/Products/Applications/Ping Warden.app/" "PingWarden/build/Ping Warden.app/"`
3. `cd PingWarden/PingWarden && bash notarize.sh X.Y.Z` (optional — standalone notarize test; `release.sh` runs it internally)
4. `bash release.sh X.Y.Z ../../RELEASE_NOTES.md` — runs `notarize.sh` (re-sign + notarize app + DMG + staple), signs the Sparkle update, creates the GitHub release, uploads dSYMs to Sentry, **and pushes the gh-pages `appcast.xml` automatically**.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oliverames/ping-warden](https://github.com/oliverames/ping-warden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
