---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**CloudTunnels** — bundle ID `com.fourninecloud.cloud-tunnels`, SwiftPM target `CloudTunnels`. Swift 5.9 / macOS 13+. A native menu-bar app plus a `ctun` CLI for managing port-forwarding tunnels across four providers: **GCP IAP**, **AWS SSM**, **Cloud SQL Auth Proxy**, and **SSH** (with optional kubeconfig auto-patching for private GKE / Loft vCluster workflows). Originally named `GCPIAPTunnel`; `ConfigStore` chains migration from `~/Library/Application Support/GCPIAPTunnel/config.json` → `~/.gcp-iap-tunnels/config.json` so existing users upgrade cleanly.

## Commands

All Makefile targets run from the package root.

```bash
make test            # swift test (XCTest, 365 tests)
make build           # swift build -c release --arch arm64 --arch x86_64
make app             # build + assemble build/CloudTunnels.app bundle
make run             # build + open the .app
make install         # copy the .app to /Applications and strip quarantine
make cli             # build release ctun only
make install-cli     # install ctun to /usr/local/bin
make zip             # build/CloudTunnels.zip (universal)
make zip-arm64       # build/CloudTunnels-arm64.zip  (Apple Silicon only)
make zip-x86_64      # build/CloudTunnels-x86_64.zip (Intel only)
make zip-all         # both single-arch zips
make clean           # swift package clean + rm .build{,-arm64,-x86_64} build
```

Single test or test class via swift directly:
```bash
swift test --filter SSHLauncherTests
swift test --filter SSHLauncherTests/testGcloudIAPUpstreamWrapsSshArgs
```

Debug build (faster iteration, no universal binary):
```bash
swift build                         # debug, native arch only
swift run CloudTunnels               # run the menu-bar app from .build
swift run ctun list                  # run CLI from .build
```

**Known gotcha:** after a failed `make install`, the `.build/apple/CompilationCache.noindex/` directory may end up owned by `root:staff` (XCBuild artifact). `rm -rf .build` will fail with permission errors. Either `sudo rm -rf .build/apple` or build into `.build-universal` via `swift build -c release --arch arm64 --arch x86_64 --build-path .build-universal`.

Logs: `log stream --predicate 'subsystem == "com.fourninecloud.cloud-tunnels"' --info`

Config: `~/Library/Application Support/CloudTunnels/config.json` (legacy read-once paths, checked in order: `~/Library/Application Support/GCPIAPTunnel/config.json`, then `~/.gcp-iap-tunnels/config.json`).

## Architecture

Three SwiftPM targets in `Package.swift`:

- **`TunnelCore`** (library) — provider-agnostic models, launchers, and helpers. Pure, Sendable-friendly. All shared logic lives here so both the GUI app and the CLI can reuse it.
- **`CloudTunnels`** (executable) — SwiftUI menu-bar app. Depends on `TunnelCore`.
- **`ctun`** (executable) — ArgumentParser CLI. Reads the same `config.json` the GUI writes and dispatches tunnels through the same `LauncherFactory`.

### Provider extensibility (critical)

The system is built around a **tagged union `ProviderConfig` enum** paired with a **`TunnelLauncher` protocol**. Adding a new provider means adding a case to both plus one switch branch in each of ~6 places. The pattern has been applied four times: GCP IAP → AWS SSM → Cloud SQL Proxy → SSH. When modifying, follow the existing shape exactly.

**`Sources/TunnelCore/Models/ProviderConfig.swift`** holds:
- `TunnelProvider` enum (`.gcpIAP`, `.awsSSM`, `.cloudSQLProxy`, `.ssh`) — the discriminator used by the UI segmented control, menu-bar tabs, and `LauncherFactory`.
- Per-provider config structs (`GCPIAPConfig`, `AWSSSMConfig`, `CloudSQLProxyConfig`, `SSHConfig`).
- `ProviderConfig` tagged union with a discriminated-union `Codable` impl (`type` + `data` fields) — stable on-disk format.
- Per-case helpers: `kind`, `targetDescription`, `accountTag`.

**`Sources/TunnelCore/TunnelLauncher.swift`** defines the launcher contract:
- `executableURL()` / `executableURL(for:)` — which binary to run. SSH needs the tunnel-aware variant to pick between `/usr/bin/ssh` and `gcloud compute ssh`.
- `arguments(for:)` — argv.
- `listeningMarkers` / `authFailurePatterns` — stderr strings `TunnelProcess` scans for to transition state.
- `fallbackConnectedSeconds` — timer-based "connected" promotion when the launcher stays silent on stderr.
- `environment(for:)` — extra env vars merged into the child process (used by Cloud SQL Proxy for `CLOUDSDK_CORE_ACCOUNT`).
- `LauncherFactory.launcher(for:)` dispatches by `tunnel.provider.kind` — the only place all four cases are enumerated for instantiation.

### Runtime lifecycle

`Tunnel` (struct, `Codable`) → `TunnelManager` (@MainActor `ObservableObject`) → `TunnelProcess` (owns one `Foundation.Process`) → launcher-provided argv.

- `TunnelManager.connect(id:)` creates a `TunnelProcess`, drains its `AsyncStream<TunnelEvent>` on a background `Task`, and translates events into `statuses: [UUID: TunnelStatus]` updates that the SwiftUI views observe.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FournineCS/cloud-tunnels](https://github.com/FournineCS/cloud-tunnels) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
