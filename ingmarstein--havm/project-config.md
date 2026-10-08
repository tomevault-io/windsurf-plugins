---
trigger: always_on
description: `havm` — Zero-config CLI for running Home Assistant OS on Apple Silicon using the native Virtualization framework. macOS 15 minimum (USB accessory passthrough requires macOS 27). Swift 6.4, built with Xcode 27+ — see Build & Test.
---

# CLAUDE.md

## Project

`havm` — Zero-config CLI for running Home Assistant OS on Apple Silicon using the native Virtualization framework. macOS 15 minimum (USB accessory passthrough requires macOS 27). Swift 6.4, built with Xcode 27+ — see Build & Test.

## Build & Test

```bash
./scripts/build.sh release    # Release build: -O + strip → ~2.1 MB binary
swift test                    # Unit tests (HavmCoreTests)
./.build/release/havm run     # Run the VM (blocks; Ctrl+C to stop)
./.build/release/havm run --console  # Interactive serial console (hvc0)
```

Binary size is reduced via `strip` (removes ~2.4 MB of symbol tables from LINKEDIT)
before codesigning. Default `-O` is kept — `-Osize` only saves ~300 KB more.

Building needs Xcode 27+ for the Swift 6.4 toolchain and the macOS 27 SDK. The
deployment target is macOS 15 and the macOS 27 APIs are weakly linked, so the
binary still runs on macOS 15 even though the host (and GitHub's `xcode-27`
image, itself macOS 27/arm64) does not. `scripts/select-xcode.sh` picks the
toolchain by the version each bundle reports — `$DEVELOPER_DIR` if it qualifies,
else the newest stable install, else the newest beta — and both workflows plus
`publish.sh` use it.

## Release Process

1. Bump `HavmVersion.current` in `Sources/Havm/main.swift` (CI also auto-bumps from tag).
2. Dry-run on a branch to exercise signing and notarization without publishing:
   `gh workflow run release.yml --ref <branch>` — a dispatch never creates a
   release, it uploads `havm.zip` as an artifact instead.
3. Tag: `git tag -a v0.1.4 -m "v0.1.4" && git push --tags`
4. CI picks up the `v*` tag, builds + notarizes, publishes a GitHub release with
   `gh release create --generate-notes`. The auto-generated notes are a starting
   point — edit the release on GitHub to add a curated changelog. To retry a
   failed tag run, re-run it from the Actions UI (keeps the original push event).

The release job runs on GitHub's `xcode-27` image, where the runner is
ephemeral and carries no signing credentials, so every credential arrives as a
secret in that run:

| Secret | Contents |
|---|---|
| `DEVELOPER_ID_CERT` | base64 of the Developer ID Application `.p12` |
| `DEVELOPER_ID_PASSWORD` | that `.p12`'s password |
| `PROVISIONING_PROFILE` | base64 of the `HAVM Developer ID` profile |
| `NOTARY_KEY` | App Store Connect API key (`.p8`) |

Plus the `DEVELOPMENT_TEAM`, `DEVELOPER_ID`, `NOTARY_KEY_ID`, and `NOTARY_ISSUER`
variables. `PROVISIONING_PROFILE` is load-bearing: without the embedded profile
the restricted entitlements (`accessory-access.usb`, `vm.networking`) still sign
but are ignored at runtime, so the release loses USB passthrough and bridge
networking silently. A "Verify signature" step fails the job if the profile or
any entitlement is missing — do not remove it. The `.p12` is imported into a
throwaway keychain, since the image's login keychain password is unknown and
`set-key-partition-list` needs the *keychain* password, not the `.p12`'s.

## Architecture

```
Havm (CLI, AsyncParsableCommand)
├── RunCommand       → HAOSSetup → VMController → ServiceRuntime
├── CleanupCommand   → FileManager
├── ImportUTMCommand → UTMImport
└── VersionCommand

HavmCore
├── Config           YAML config, paths, parsing (Yams)
├── HAOSSetup        GitHub release fetch, download .img.xz, xz decompress (CXZ/libzma),
│                    copy+resize disk, SSH CONFIG disk
├── VMController     VZEFIBootLoader + VZEFIVariableStore, storage, network, USB,
│                    machine identifier persistence, console mode
│                    (VZVirtioConsoleDeviceSerialPortConfiguration,
│                    VZFileHandleSerialPortAttachment),
│                    @MainActor on start()
├── ServiceRuntime   SIGTERM/SIGINT → SSH shutdown (port 22222/22) →
│                    force-stop fallback, DHCP lease guest IP detection,
│                    console mode: raw terminal, skip SIGINT, restore on exit,
│                    AAUSBAccessoryListener + VZUSBPassthroughDevice for USB
├── CONFIGDiskBuilder MBR + FAT16 with VFAT LFN, volume label "CONFIG",
│                    authorized_keys file — HA OS auto-imports for SSH
├── Metrics           Prometheus metrics: MetricsServer (NWListener HTTP,
│                    TCP and/or Unix socket), bootstrap, process gauges
└── Config/MemorySize Human-readable sizes ("4 GiB" → bytes)

CXZ (C target)
└── xz_decompress    Statically links liblzma (-llzma) for XZ decompression (no external tools)
```

## Key Design Decisions

- **VZEFIBootLoader** — boots directly from GPT disk via UEFI. No kernel extraction, no kernel command line, no initrd. Just point at the disk image.
- **Bridge networking by default** — LAN-reachable IP for Home Assistant discovery. Falls back to NAT at runtime if the binary lacks the `com.apple.vm.networking` entitlement (e.g. self-compiled). Explicit `network.type: nat` available for manual override.
- **@MainActor on VM start** — `VZVirtualMachine.start()` has `dispatch_assert_queue` requiring the main queue.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IngmarStein/havm](https://github.com/IngmarStein/havm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
