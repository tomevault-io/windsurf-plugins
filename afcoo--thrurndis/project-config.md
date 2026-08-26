---
trigger: always_on
description: generates the guest `wg0.conf` and renders the host configuration for manual
---

# AGENTS.md

This repository is a macOS 27+ USB RNDIS tethering VM project. Future agents
should read this file before the README and treat the current
WireGuard-over-VZNAT architecture as the baseline.

## Project Shape

- The app is a menu-bar utility with an AppKit `NSStatusItem`, not a Dock app or
  CLI `main.swift` entrypoint. It has no primary `WindowGroup`; SwiftUI provides
  the Settings scene, while a small AppKit window controller presents
  first-run onboarding.
- The Xcode project is `ThruRNDIS.xcodeproj`.
- The main app target is `ThruRNDIS` and builds a macOS app bundle.
- `ThruRNDISWireGuardNetworkExtension` is a WireGuardKit-backed Network System
  Extension embedded under `Contents/Library/SystemExtensions`. The app manages
  one host packet-tunnel profile and session, but does not inspect or relay
  packet payloads itself.
- `ThruRNDISPrivilegedHelper` is a small command-line privileged helper embedded
  at `Contents/MacOS/ThruRNDISPrivilegedHelper`, with its launchd property list
  at `Contents/Library/LaunchDaemons/ThruRNDISPrivilegedHelper.plist`. The app
  initially registers it with `SMAppService.daemon` only after an explicit
  request in the onboarding permissions page or Dummy Ethernet Settings tab.
  A registered helper is automatically replaced when its build version differs
  from the current app build. The app bundle is the helper executable's only
  source; never copy it to
  `/Library/PrivilegedHelperTools` or maintain a versioned system-path copy.
  This project does not use DriverKit.
- Linux assets are not bundled with the app. The shared onboarding and Settings
  flow presents `Download & Install Latest` while assets are unconfigured and
  `Check & Install Latest` once a valid selection is ready. The app downloads
  the exact `vm_assets.zip` and `SHA256SUMS` attachments from the latest published
  [Afcoo/ThruRNDIS_VM_Assets Release](https://github.com/Afcoo/ThruRNDIS_VM_Assets/releases),
  verifies and installs them in Application Support, and activates the managed
  release. Manual download, checksum verification, extraction, and folder
  selection remain a fallback. An optional raw scratch disk is user-managed
  separately. Do not direct users to build VM assets locally from this
  repository.
- The VM boots with the kernel image and initial ramdisk (initramfs) contained
  in the `vm_assets.zip` release artifact. After extraction and selection,
  `VMConfigurationFactory` passes those files from the selected `vm_assets`
  folder to `VZLinuxBootLoader` as its kernel and `initialRamdiskURL`.

## Architecture

- `TetheringStore` is the app-facing facade for reset ordering, onboarding
  presentation/listener coordination, and general VM, USB, and WireGuard
  commands. It adapts VM and USB callbacks into observable presentation state
  and forwards cross-feature events to `TetheringWorkflowCoordinator`, which
  owns only the serialized USB approval, VM preparation, passthrough, and
  optional WireGuard request workflow. `ManagedWireGuardConnectionCoordinator`
  separately owns the Dummy Ethernet preparation, WireGuard tunnel wait, cleanup,
  cancellation, and stale-operation protection used by app-managed connections.
  State that can be observed independently lives in child stores. `EventLogStore`
  owns the bounded in-memory app event log and screen filtering, while
  `EventLogFileStore` serially persists every event under Application Support
  with 10 MiB or 24-hour session-file rotation and seven-day retention.
  `ConsoleSessionStore`
  owns only VM serial-console output and endpoint scanning, `USBSessionStore`
  owns the atomic USB UI snapshot plus USB attachment prompt queue, de-duplication,
  and VM-asset deferral, `VMConfigurationStore`
  owns persisted VM settings including the optional scratch disk,
  `WireGuardSessionStore` owns WireGuard tunnel/System Extension presentation state,
  the USB-triggered WireGuard connection prompt, WireGuard inputs, validation,
  and configuration readiness, and
  `AppPreferencesStore` owns onboarding completion, USB/WireGuard preferences,
  WireGuard Manual Configuration Mode, and Launch at Login state. WireGuard
  Manual Configuration Mode keeps guest WireGuard configuration generation and host configuration
  export available, but disables app-managed host WireGuard connections and
  Dummy Ethernet. VM lifecycle work belongs in `VMCoordinator`; USB
  AccessoryAccess selection and passthrough policy belong in
  `USBAccessoryCoordinator`.
- `AppDelegate` is the composition root. It owns one shared
  `VMAssetWorkflowCoordinator`, constructs the VM, USB, and WireGuard adapters
  and the independently injected child state stores, and injects them into one shared `TetheringStore`,
  requests AccessoryAccess monitoring at app launch, and passes the same objects
  to onboarding, Settings, and the menu bar. Before each listener start in normal
  app-managed mode, `TetheringStore` requires completed onboarding, valid VM Assets,
  an active Network Extension, and the current Dummy Ethernet helper. Manual
  Configuration Mode requires only completed onboarding and valid VM Assets;
  it must not require Network Extension activation or privileged-helper installation.
  Views observe the narrowest child

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Afcoo/ThruRNDIS](https://github.com/Afcoo/ThruRNDIS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
