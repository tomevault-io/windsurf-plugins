---
trigger: always_on
description: This is an embedded controller (EC) services workspace — a collection of `no_std` Rust crates implementing hardware-agnostic business logic for embedded controllers. Services glue together MCU HALs (via `embedded-hal` traits), peripheral drivers, and EC subsystem abstractions (sensors, batteries, fans, USB-PD, etc.) using the Embassy async runtime.
---

# Copilot Instructions for embedded-services

## Overview

This is an embedded controller (EC) services workspace — a collection of `no_std` Rust crates implementing hardware-agnostic business logic for embedded controllers. Services glue together MCU HALs (via `embedded-hal` traits), peripheral drivers, and EC subsystem abstractions (sensors, batteries, fans, USB-PD, etc.) using the Embassy async runtime.

## Build, Test, and Lint

Toolchain: Rust 1.90 (`rust-toolchain.toml`), edition 2024. Targets: `x86_64-unknown-linux-gnu` (std/testing) and `thumbv8m.main-none-eabihf` (ARM Cortex-M33).

```shell
# Format
cargo fmt --check

# Lint (all feature combos, both targets)
cargo hack --feature-powerset --mutually-exclusive-features=log,defmt,defmt-timestamp-uptime clippy --locked --target x86_64-unknown-linux-gnu
cargo hack --feature-powerset --mutually-exclusive-features=log,defmt,defmt-timestamp-uptime clippy --locked --target thumbv8m.main-none-eabihf

# Test (workspace, host target only)
cargo test --locked

# Test a single crate
cargo test --locked -p partition-manager

# Test a single test function
cargo test --locked -p partition-manager test_name

# Lint test code
cargo clippy --locked --tests

# Check docs
cargo doc --no-deps -F log --locked
cargo doc --no-deps -F defmt --locked

# Unused dependency check
cargo machete

# Dependency license/advisory/audit checks
cargo deny check --all-features --locked
cargo vet --locked
```

The `examples/` directory contains separate workspaces (excluded from the root workspace). Build/lint them independently:

```shell
# ARM board examples
cd examples/rt685s-evk && cargo clippy --target thumbv8m.main-none-eabihf --locked
cd examples/rt633 && cargo clippy --target thumbv8m.main-none-eabihf --locked
# Std examples
cd examples/std && cargo clippy --locked
```

## Architecture

> **Note:** The `v0.2.0` branch is the target for new development and
> contains the latest service patterns. Some services on `main` still
> use older patterns (e.g., `comms::Endpoint`, `MailboxDelegate`,
> `OnceLock` singletons) that are being phased out. When adding or
> modifying services, follow the patterns described below and on
> `v0.2.0`. See also [`docs/api-guidelines.md`](../docs/api-guidelines.md)
> for detailed rationale.

### Service Pattern

Services implement the `odp_service_common::runnable_service::Service<'hw>` trait, which enforces a consistent structure:

1. **`Resources`** — caller-allocated state (stored in a `StaticCell`), not an internal `OnceLock` singleton
2. **`new(resources, params) -> (Self, Runner)`** — constructor returns a control handle and a `Runner`
3. **`Runner`** — implements `ServiceRunner` with a single `run(self) -> !` method that drives the service's async event loop
4. **`spawn_service!`** macro — handles boilerplate: allocates `Resources` in a `StaticCell`, calls `new()`, spawns the `Runner` on an Embassy executor

```rust
// Typical service using the Service trait
#[derive(Default)]
pub struct Resources<'hw> {
    inner: Option<ServiceInner<'hw>>,
}

pub struct MyService<'hw> { /* control handle */ }
pub struct Runner<'hw> { /* holds refs into Resources */ }

impl<'hw> Service<'hw> for MyService<'hw> {
    type Resources = Resources<'hw>;
    type Runner = Runner<'hw>;
    type InitParams = MyInitParams<'hw>;
    type ErrorType = MyError;

    async fn new(
        resources: &'hw mut Self::Resources,
        params: Self::InitParams,
    ) -> Result<(Self, Self::Runner), Self::ErrorType> {
        // ...
    }
}
```

Key principles (from API guidelines):

- **No `'static` references** — use generic `'hw` lifetimes for testability
- **External memory allocation** — callers provide `Resources`, no internal `static OnceLock` singletons
- **Trait-based public APIs** — runtime interfaces live in standalone `-interface` crates (e.g., `battery-service-interface`) for mockability and customizability

### Communication (IPC)

Services use a variety of async IPC mechanisms from `embassy-sync` and `embedded_services`:

- **`embassy_sync::channel::Channel`** — bounded async MPSC channels for command/response flows
- **`embassy_sync::signal::Signal`** — single-value async notifications
- **`embedded_services::ipc::deferred`** — request/response channels where the caller awaits a reply
- **`embedded_services::broadcaster`** — publish/subscribe pattern for event fan-out
- **`embedded_services::relay`** — relay service pattern for MCTP-based request/response dispatch with direct async calls

### Composition

At the top level, an EC is composed by spawning service tasks on an Embassy executor, using the `spawn_service!` macro:

```rust
let my_service = spawn_service!(spawner, MyService, my_init_params)?;
```

### Core Utilities (embedded-service crate)

- **`GlobalRawMutex`**: `ThreadModeRawMutex` on ARM bare-metal, `CriticalSectionRawMutex` on RISC-V bare-metal and in tests/std

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenDevicePartnership/embedded-services](https://github.com/OpenDevicePartnership/embedded-services) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
