---
trigger: always_on
description: enables detailed discovery, resolve, registration, auto-refresh and network path
---

# AGENTS.md

This is a native macOS menu bar app named Beacon. It re-broadcasts Bonjour/mDNS
services that are visible to the Mac but not visible to the rest of the LAN,
with Docker and OrbStack container services as the motivating case. The common
example is Home Assistant or a HomeKit bridge running in a container: the Mac can
resolve the service, but other devices cannot discover it until Beacon advertises
it through the host mDNSResponder.

Beacon is intentionally small and native:

- SwiftUI app, macOS 14 minimum.
- Pure Swift wrappers around the system `dns_sd` API from mDNSResponder.
- No Python, no `dns-sd` subprocesses.
- Xcode project generated from `project.yml` by XcodeGen.
- Settings and whitelist are persisted as JSON in
  `~/Library/Application Support/Beacon/`.

## Repository Map

- `Beacon/BeaconApp.swift`: app entry point, menu bar extra, main window scene,
  and shared observable object setup.
- `Beacon/Engine/`: DNS-SD wrappers and runtime broadcasting orchestration.
- `Beacon/Model/`: persisted settings, whitelist entries, service type catalog,
  and JSON store.
- `Beacon/UI/`: SwiftUI screens for Status, Discovery, Whitelist, Settings, and
  About.
- `Beacon/Assets.xcassets/`: app icons and template menu bar icons.
- `Beacon/Info.plist`: local network usage text and Bonjour service type list.
- `Beacon/Beacon.entitlements`: network client/server entitlements.
- `SelfTest/`: command line harness for browsing, resolving, registering, and
  mirroring services through the same engine components.
- `project.yml`: source of truth for the Xcode project.
- `scripts/release.sh`: Developer ID release build, DMG packaging, optional
  notarization and stapling.
- `.env.example`: template for local signing/notarization variables.

Generated or local files may exist in a checkout but should not be treated as
source of truth: `Beacon.xcodeproj/`, `build/`, `dist/`, `.env`, `.DS_Store`,
`xcuserdata/`, and local tool settings such as `.claude/settings.local.json`.

## How Beacon Works

The core flow is:

1. Discover service instances with `DNSServiceBrowse`.
2. Resolve a selected or whitelisted instance with `DNSServiceResolve`.
3. Re-register that service with `DNSServiceRegister`.
4. Keep the DNS service ref alive while broadcasting.
5. Deallocate the ref to withdraw the advertisement.

The important behavior is in `Beacon/Engine/ServiceRegistrar.swift`: registration
passes `host = nil` to `DNSServiceRegister`. That makes the SRV target point at
the local Mac, matching `dns-sd -R` behavior. Do not "fix" this by passing the
resolved container host. Beacon deliberately preserves the original port and TXT
record while advertising from the host.

Port values from `DNSServiceResolve` are in network byte order. `ResolvedService`
stores the raw network-order port for registration and exposes `displayPort` for
UI. Keep that distinction intact.

TXT records are round-tripped as raw bytes for mirrored services. The `TXT`
helpers in `DNSSD.swift` are for display and for fresh self-test registrations.
Avoid replacing the raw mirrored TXT data with parsed/re-encoded data unless that
is the explicit feature being built.

## Runtime Architecture

`Store` is the persisted source of truth. It is `@Observable` and `@MainActor`,
and owns:

- `AppSettings`, saved to `settings.json`.
- `[WhitelistEntry]`, saved to `whitelist.json`.

`BroadcastEngine` is the transient runtime coordinator. It is also
`@Observable` and `@MainActor`, and owns:

- per-entry run states keyed by `WhitelistEntry.ID`;
- active `ServiceRegistrar` instances;
- desired-running IDs;
- advertised names, used to flag Beacon's own re-broadcasts in Discovery;
- auto-refresh timer;
- `NWPathMonitor` refresh on network reconnection.

The engine resolves each entry by trying the entry's `preferredServiceType`
first, then the selected service types from settings. On a successful resolve it
records the last host, port, service type, and date back into the store.

`ServiceBrowser`, `ServiceResolver`, and `ServiceRegistrar` each own DNS-SD refs
and dispatch queues. Be careful with lifetime management: callbacks use retained
contexts, refs are deallocated on the queue used by callbacks, and continuations
must resume exactly once.

## UI Architecture

Beacon is menu-bar-first:

- `MenuBarExtra` hosts `StatusPanel`, the primary status and start/stop surface.
- The `Window("Beacon", id: WindowID.main)` hosts the configuration UI.
- `MainWindow` uses `NavigationSplitView` over `MainTab`.
- `AppRouter` lets the menu bar panel deep-link to Discovery, Whitelist,
  Settings, or About in the main window.

Screen responsibilities:

- `DiscoveryView`: live browsing over `store.settings.discoveryServiceTypes`,
  resolving the selected row, and adding services to the whitelist.
- `WhitelistView`: manual add, enable/pause, reorder, delete, and per-entry
  run/stop.
- `SettingsView`: service groups, custom service types, launch at login,
  start delay, auto-refresh, verbose setting, legacy whitelist import, and the
  hidden icon setting after unlock.
- `StatusPanel`: compact menu bar dropdown for enabled services, run state, and
  Start All/Stop All.
- `AboutView`: version display, links, app icon preview, and hidden icon unlock.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lpbas/Beacon](https://github.com/lpbas/Beacon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
