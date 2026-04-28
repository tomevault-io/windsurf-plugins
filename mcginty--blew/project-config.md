---
trigger: always_on
description: A cross-platform BLE (Bluetooth Low Energy) library for Rust, providing both Central and Peripheral roles with platform backends for Apple (CoreBluetooth via objc2), Linux (BlueZ via bluer), and Android (JNI + Kotlin).
---

# blew — Project Guide for Claude

## What this is

A cross-platform BLE (Bluetooth Low Energy) library for Rust, providing both Central and Peripheral roles with platform backends for Apple (CoreBluetooth via objc2), Linux (BlueZ via bluer), and Android (JNI + Kotlin).

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
├── gatt/
│   ├── props.rs                  # CharacteristicProperties, AttributePermissions (bitflags)
│   └── service.rs                # GattService, GattCharacteristic, GattDescriptor
├── central/
│   ├── mod.rs                    # Central<B>  (default B = PlatformCentral)
│   │                             #   + CentralEvents<S> wrapper type
│   ├── types.rs                  # CentralEvent, ScanFilter, WriteType
│   └── backend.rs                # CentralBackend sealed trait (RPITIT, no async_trait)
├── peripheral/
│   ├── mod.rs                    # Peripheral<B> (default B = PlatformPeripheral)
│   │                             #   + PeripheralEvents<S> wrapper type
│   ├── types.rs                  # PeripheralEvent (!Clone), ReadResponder, WriteResponder,
│   │                             #   AdvertisingConfig
│   └── backend.rs                # PeripheralBackend sealed trait
├── l2cap/
│   ├── mod.rs                    # L2capChannel stub (AsyncRead + AsyncWrite)
│   └── types.rs                  # Psm(u16) newtype
├── platform/
│   ├── mod.rs                    # #[cfg] type aliases: PlatformCentral, PlatformPeripheral
│   ├── apple/
│   │   ├── central.rs            # AppleCentral — full CoreBluetooth implementation
│   │   └── peripheral.rs         # ApplePeripheral — full CoreBluetooth implementation
│   ├── linux/
│   │   ├── central.rs            # LinuxCentral — full bluer/BlueZ implementation
│   │   └── peripheral.rs         # LinuxPeripheral — full bluer/BlueZ implementation
│   └── android/
│       ├── mod.rs                # Exports + init_jvm re-export
│       ├── jni_globals.rs        # OnceLock<JavaVM>, init_jvm(), jvm()
│       ├── central.rs            # AndroidCentral — JNI bridge to BleCentralManager.kt
│       ├── peripheral.rs         # AndroidPeripheral — JNI bridge to BlePeripheralManager.kt
│       └── jni_hooks.rs          # #[unsafe(no_mangle)] extern "C" JNI callbacks
├── testing.rs                    # In-memory mock backends (feature = "testing")
└── util/
    ├── event_fanout.rs           # EventFanout<E: Clone> — mpsc fan-out for CentralEvent
    └── request_map.rs            # RequestMap<V> — thread-safe pending request/response coupling
```

## Public API pattern

```rust
// Both roles are independent; use only what you need.
let central: Central = Central::new().await?;   // explicit type required — see below
let peripheral: Peripheral = Peripheral::new().await?;

let mut events = central.events();   // returns CentralEvents<_> (impl Stream)
use tokio_stream::StreamExt as _;
while let Some(ev) = events.next().await { ... }
```

**`let central: Central` is required.** Rust's default type-parameter inference does not kick in for method calls; without the explicit annotation the compiler fails with E0283. Same for `Peripheral`.

## Apple backend design (`platform/apple/`)

**Threading model:**
- Each manager (`CBCentralManager`, `CBPeripheralManager`) is initialized with a dedicated GCD serial queue via `initWithDelegate_queue(Some(&queue))`.
- All CB delegate callbacks fire exclusively on that queue.
- Tokio tasks call CB methods directly from the thread pool; CoreBluetooth is documented thread-safe on macOS 10.15+ / iOS 13+.
- Results flow back to Tokio via `tokio::sync::oneshot` channels (set in the delegate callback, awaited in the async method).

**Key patterns:**

```rust

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mcginty) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-17 -->
