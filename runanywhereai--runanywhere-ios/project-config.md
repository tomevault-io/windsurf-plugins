---
trigger: always_on
description: One SwiftUI target, `RunAnywhereAI`, shipping to both the App Store and the Mac App Store,
---

# AGENTS.md RunAnywhereAI for iOS and macOS

One SwiftUI target, `RunAnywhereAI`, shipping to both the App Store and the Mac App Store,
plus a keyboard extension and a Live Activity widget. It consumes the RunAnywhere SDK from
its published GitHub release. The app was extracted from the `runanywhere-sdks` monorepo at
release 0.20.17 with history preserved, so every path below is relative to this repository's
root, not to a monorepo `examples/` directory.

---

## Build and run

```bash
# Simulator
./scripts/build_and_run_ios_sample.sh simulator "iPhone 16 Pro"

# Physical device
./scripts/build_and_run_ios_sample.sh device

# Native macOS
./scripts/build_and_run_ios_sample.sh mac
```

`open RunAnywhereAI.xcodeproj` works too; SwiftPM resolves the SDK on open. `./scripts/verify.sh`
is the local gate (resolve plus full `xcodebuild`), and `./scripts/smoke.sh` greps the sources
for SDK call patterns without compiling.

Logs: `log stream --predicate 'subsystem CONTAINS "com.runanywhere"' --info --debug` on
simulator and Mac, `idevicesyslog | grep "com.runanywhere"` on device.

---

## App Store release

`docs/RELEASE_INSTRUCTIONS.md` carries the full flow. The packaged XCFrameworks already
declare the iOS 17.5 deployment floor, so release archives validate that metadata rather
than mutating it after the build.

### Native symbol gate (iOS)

Release stripping or a stale XCFramework can drop a Swift-facing native symbol and produce a
runtime startup failure such as `Native proto ABI is not exported by the linked RACommons
binary: rac_sdk_init_phase1_proto`. Every archive must therefore keep the export surface
intact:

- `RunAnywhereExportedSymbols.txt` contains `_rac_*` and `_ra_mlx_*`.
- The Release app target links with `-all_load`.
- The Release app target passes `-Wl,-exported_symbols_list,$(SRCROOT)/RunAnywhereExportedSymbols.txt`.
- The Release app target sets `STRIP_STYLE = non-global` so `dlsym` still resolves after
  archive post-processing.
- `RunAnywhereExportedSymbols.txt` is not bundled into app resources.

From the repository root:

```bash
# 1. Build the final release inputs.
xcodebuild \
  -project RunAnywhereAI.xcodeproj \
  -scheme RunAnywhereAI \
  -configuration Release \
  -destination 'generic/platform=iOS' \
  -skipPackagePluginValidation \
  -jobs "$(sysctl -n hw.logicalcpu)" \
  build

# 2. Archive directly into Xcode Organizer's archive folder.
ARCHIVE_DIR="$HOME/Library/Developer/Xcode/Archives/$(date +%Y-%m-%d)"
ARCHIVE="$ARCHIVE_DIR/RunAnywhereAI-$(date +%Y%m%d-%H%M%S).xcarchive"
mkdir -p "$ARCHIVE_DIR"
xcodebuild \
  -project RunAnywhereAI.xcodeproj \
  -scheme RunAnywhereAI \
  -configuration Release \
  -destination 'generic/platform=iOS' \
  -archivePath "$ARCHIVE" \
  -allowProvisioningUpdates \
  -skipPackagePluginValidation \
  -jobs "$(sysctl -n hw.logicalcpu)" \
  archive

open -a Xcode "$ARCHIVE"
```

Then audit the archived binary:

```bash
APP="$ARCHIVE/Products/Applications/RunAnywhereAI.app"
BIN="$APP/RunAnywhereAI"

nm -gjU "$BIN" 2>/dev/null \
  | rg '^_(rac|ra_mlx)_' \
  | sed 's/^_//' \
  | sort -u > /tmp/runanywhere_archive_exported_symbols.txt

# `swift package resolve` places the SDK sources here. Xcode archives resolve
# into DerivedData instead, so override with
# SDK_CHECKOUT=<path-to-runanywhere-sdks-checkout> when auditing those.
# The 0.20.17 restructure moved the Swift SDK from sdk/runanywhere-swift/ to
# bindings/swift/; an older checkout still uses the former.
SDK_CHECKOUT="${SDK_CHECKOUT:-.build/checkouts/runanywhere-sdks}"
SRC_DIRS=(
  "$SDK_CHECKOUT/bindings/swift/Sources/RunAnywhere"
  "$SDK_CHECKOUT/bindings/swift/Sources/LlamaCPPRuntime"
  "$SDK_CHECKOUT/bindings/swift/Sources/ONNXRuntime"
  "$SDK_CHECKOUT/bindings/swift/Sources/MLXRuntime"
)

rg -No '"(rac|ra_mlx)_[A-Za-z0-9_]+"' "${SRC_DIRS[@]}" --glob '*.swift' \
  | perl -ne 'while (/"((?:rac|ra_mlx)_[A-Za-z0-9_]+)"/g) { print "$1\n" }' \
  | sort -u > /tmp/runanywhere_expected_swift_native_symbols.from_strings

# Declared only inside a build-configuration guard this archive does not compile.
# The rg pass above is a plain text scan and does not evaluate `#if`, so without
# this filter the gate fails on every good archive over a symbol that is
# CORRECTLY absent. `ra_mlx_metal_resource_anchor` lives in MLXRuntime/MLX.swift
# under `#if RUNANYWHERE_MLX_DISTRIBUTION`, set only for the CocoaPods
# distribution build; a SwiftPM archive never compiles it.
# Extend this list ONLY for a symbol confirmed guarded out of this configuration.
PACKAGING_ONLY_SYMBOLS=(
  ra_mlx_metal_resource_anchor
)

{
  cat /tmp/runanywhere_expected_swift_native_symbols.from_strings
  printf '%s\n' \
    rac_proto_buffer_free \
    rac_backend_llamacpp_register \
    rac_backend_llamacpp_unregister \
    rac_backend_onnx_register \
    rac_backend_onnx_unregister \
    rac_plugin_entry_sherpa \
    rac_plugin_register \
    rac_plugin_unregister \
    rac_backend_mlx_register \
    rac_backend_mlx_unregister \
    rac_mlx_set_callbacks \
    ra_mlx_register_runtime \
    ra_mlx_runtime_is_available \
    ra_mlx_runtime_is_registered \
    ra_mlx_unregister_runtime
} | sort -u \
  | grep -vxF "$(printf '%s\n' "${PACKAGING_ONLY_SYMBOLS[@]}")" \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RunanywhereAI/runanywhere-ios](https://github.com/RunanywhereAI/runanywhere-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
