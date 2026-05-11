---
trigger: always_on
description: A cross-platform BLE (Bluetooth Low Energy) library for Rust, providing both Central and Peripheral roles with platform backends for Apple (CoreBluetooth via objc2), Linux (BlueZ via bluer), and Android (JNI + Kotlin).
---

# blew — Project Guide for Codex

## What this is

A cross-platform BLE (Bluetooth Low Energy) library for Rust, providing both Central and Peripheral roles with platform backends for Apple (CoreBluetooth via objc2), Linux (BlueZ via bluer), and Android (JNI + Kotlin).

## Why this exists

`blew` was extracted from [iroh-ble-transport](https://github.com/mcginty/iroh-ble-transport) (local clone: `~/git/iroh-ble-transport`) — that project is the primary driver for the API shape, L2CAP focus, and cross-platform requirements. When in doubt about a design decision, check what `iroh-ble-transport` needs. Many of the quirks handled here (e.g., post-connect MTU stability, L2CAP ergonomics) came directly from issues encountered there.

## Project goals

- L2CAP is a first-class transport, not an edge feature.
- Designed to run with as many concurrent L2CAP channels as the device and OS will allow.
- Backend-owned transports with explicit close/shutdown. Shared event-loop / reactor threads (Apple `L2capReactor`, Linux BlueZ, Android Kotlin coroutines) instead of per-channel worker threads. When a short-term fix is tempting, bias toward the reactor-shaped architecture anyway.

## Commands

Uses [mise](https://mise.jdx.dev) for task management. Run `mise tasks` for the full list.

```sh
mise run build                           # build all crates
mise run test                            # run all tests (nextest)
mise run lint                            # clippy
mise run fmt                             # format
mise run fmt:check                       # check formatting
mise run deny                            # license/vulnerability audit
cargo run --example scan -p blew         # scan for 10s
cargo run --example advertise -p blew    # advertise GATT service
```

## Style

- No comments unless the logic is non-obvious. Don't add doc comments to code you didn't change.
- Don't add features, refactor, or "improve" code beyond what was asked.
- Clippy pedantic is enabled (`pedantic = "warn"` in blew). Fix warnings, don't suppress them unless there's a good reason.
- Test with nextest: `cargo nextest run --workspace`.
- **Update `CHANGELOG.md` alongside user-visible changes.** New entries go under
  `## [Unreleased]` (creating that heading if missing) in the `Added` / `Changed` /
  `Removed` / `Fixed` buckets. Breaking changes must also include a before/after
  snippet in the upgrade guide at the bottom of the file. Pure-internal refactors
  that don't alter public API or observable behavior can skip the changelog.

## Key dependencies

| Crate | Role |
|-------|------|
| `objc2` / `objc2-core-bluetooth` | Apple backend (CoreBluetooth) |
| `bluer 0.17` | Linux backend (BlueZ D-Bus bindings) |
| `jni 0.22` | Android backend (JNI bridge) |
| `tokio 1` | Async runtime |

## Module structure

```
crates/blew/src/
├── lib.rs                        # pub use re-exports; top-level doc example
├── error.rs                      # BlewError (typed enum), BlewResult<T>
├── types.rs                      # DeviceId (Display + as_str()), BleDevice
├── testing.rs                    # In-memory mock backends (feature = "testing")
├── gatt/
│   ├── props.rs                  # CharacteristicProperties, AttributePermissions (bitflags)
│   └── service.rs                # GattService, GattCharacteristic, GattDescriptor
├── central/
│   ├── mod.rs                    # Central<B>  (default B = PlatformCentral)
│   ├── types.rs                  # CentralEvent, ScanFilter, WriteType
│   └── backend.rs                # CentralBackend sealed trait (RPITIT, no async_trait)
├── peripheral/
│   ├── mod.rs                    # Peripheral<B> (default B = PlatformPeripheral)
│   │                             #   + state_events() / take_requests() accessors
│   ├── types.rs                  # PeripheralStateEvent (Clone), PeripheralRequest (!Clone),
│   │                             #   ReadResponder, WriteResponder, AdvertisingConfig
│   └── backend.rs                # PeripheralBackend sealed trait
├── l2cap/
│   ├── mod.rs                    # L2capChannel (AsyncRead + AsyncWrite) with close hook
│   └── types.rs                  # Psm(u16) newtype
├── platform/
│   ├── mod.rs                    # #[cfg] type aliases: PlatformCentral, PlatformPeripheral
│   ├── apple/
│   │   ├── central.rs            # AppleCentral — full CoreBluetooth implementation
│   │   ├── peripheral.rs         # ApplePeripheral — full CoreBluetooth implementation
│   │   └── l2cap.rs              # L2capReactor — single-thread NSRunLoop for all channels
│   ├── linux/
│   │   ├── central.rs            # LinuxCentral — full bluer/BlueZ implementation
│   │   ├── peripheral.rs         # LinuxPeripheral — full bluer/BlueZ implementation
│   │   └── l2cap.rs              # bluer::l2cap::Stream → L2capChannel bridge
│   └── android/
│       ├── mod.rs                # Exports + init_jvm re-export
│       ├── jni_globals.rs        # OnceLock<JavaVM>, init_jvm(), jvm()
│       ├── central.rs            # AndroidCentral — JNI bridge to BleCentralManager.kt

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mcginty/blew](https://github.com/mcginty/blew) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
