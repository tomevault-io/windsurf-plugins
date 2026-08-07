---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

usbipd-mac is a macOS USB/IP protocol implementation for sharing USB devices over IP networks. The project is built using Swift Package Manager and targets macOS 11+.

**Main Repository**: https://github.com/beriberikix/usbipd-mac  
**Homebrew Tap Repository**: https://github.com/beriberikix/homebrew-usbipd-mac

### Current state — what works and what does not

**Works, verified against hardware.** Devices macOS has *not* bound a driver to are
served end to end: enumeration, string descriptors, control transfers, and
bidirectional bulk transfers. Two device classes have been driven: a SEGGER J-Link with
probe-rs, which read the probe's VTref over the wire and behaved exactly as it does
connected directly, and a Pixel 10a in ADB mode, which answered a CNXN with its AUTH
challenge. No System Extension and no entitlement are involved.

**A caveat that is not about claiming.** A client whose protocol keys off USB
connection or reset events may not work even on a claimable device. `adb` reports a
Pixel `offline` because the phone announces itself once per connection and macOS
already received that announcement; attaching from a client causes no bus reset the
phone can observe. Request/response devices are unaffected. See
`Documentation/development/android-adb-validation.md`.

**Does not work, and cannot be made to.** Devices macOS binds a driver to —
USB-serial, HID, mass storage, audio, cameras. `bind` refuses these up front with an
explanation naming the owner. This was measured, not assumed: `USBInterfaceOpenSeize`
returns the same `kIOReturnExclusiveAccess` as a plain open, and neither unmounting
nor ejecting releases a device. Only the DriverKit USB transport entitlements would
change it, and Apple has to grant those.

**Untested.** Interrupt endpoints (no unbound interrupt device has been available).
Isochronous is not merely untested but structurally incomplete: alternate settings are
never selected and pipes are discovered once at open, so a UVC device's isochronous
endpoints would never appear. See `Documentation/development/probe-rs-validation.md`.

The **SystemExtension** target is quarantined — roughly 20,000 lines that no shipping
path uses. `OSSystemExtensionRequest` resolves extensions inside the calling app's
bundle and requires that bundle to live in `/Applications`, so a Homebrew install
never consults it. See `Sources/USBIPDCore/SystemExtension/README.md`.

## Architecture

The project is structured as a multi-target Swift package:

### Core Targets
- **USBIPDCore**: Core USB/IP protocol implementation and device management
  - `Device/`: IOKit-based USB device discovery and monitoring
  - `Network/`: TCP server and client connection handling
  - `Protocol/`: USB/IP message encoding/decoding and request processing
- **USBIPDCLI**: Command-line interface executable (`usbipd` binary)
- **Common**: Shared utilities (logging, error handling)
- **SystemExtension**: quarantined; no shipping path activates it (see above)
- **QEMUTestServer**: QEMU validation test server

### Test Structure

`Package.swift` declares two test targets, and they are the whole suite:

- **Tests/USBIPDCoreTests/** — core protocol, device, and network tests
- **Tests/USBIPDCLITests/** — CLI behaviour

Both also compile **Tests/SharedUtilities/** via their `sources:` list.

Three further targets are declared but commented out as "temporarily disabled":
`IntegrationTests`, `SystemExtensionTests`, `QEMUIntegrationTests`. Alongside them
`Tests/TestMocks/`, `Tests/ProductionTests/` and `Tests/PerformanceTests/` are compiled
by nothing. None of it has built in about a year — do not cite coverage from it without
reviving the target first. See `Documentation/development/testing-strategy.md`.

## Development Commands

### Build
```bash
# Standard build
swift build

# Build specific product
swift build --product QEMUTestServer

# Xcode build
xcodebuild -scheme usbipd-mac build
```

### Testing

```bash
# Run the test suite
swift test --parallel

# Run one target or one test
swift test --filter USBIPDCoreTests
swift test --filter USBIPDCoreTests.USBIPProtocolTests
```

Only two test targets exist in `Package.swift`: `USBIPDCLITests` and
`USBIPDCoreTests`. There is no tiered development/CI/production test system — the
scripts that claimed to provide one filtered on target names that were never declared,
so they matched nothing and exited 0. They were removed in 2026-08 along with the CI
step that called them. `swift test` is the whole story.

Note that `swift test` needs XCTest, which ships with Xcode. A machine with only the
Command Line Tools cannot run it (`error: no such module 'XCTest'`); use CI, which runs
on `macos-latest` with full Xcode.

### Code Quality
```bash
# Run SwiftLint (strict mode like CI)
swiftlint lint --strict

# Auto-fix violations
swiftlint --fix
```

### Full CI Validation Locally
```bash
# Complete validation sequence (matches consolidated CI pipeline)
swiftlint lint --strict                      # Code quality validation
swift build --verbose                        # Build validation  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beriberikix/usbipd-mac](https://github.com/beriberikix/usbipd-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
