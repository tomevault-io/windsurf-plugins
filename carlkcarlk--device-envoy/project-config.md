---
trigger: always_on
description: This file contains both shared workspace rules and crate-specific rules for this repository.
---

# Coding Notes for Agents

This file contains both shared workspace rules and crate-specific rules for this repository.

## General Policies

- **Never silently skip required build targets in xtask/CI.** Every supported target (e.g., ESP32-C6, ESP32-S3, Pico 1, Pico 2) must be built on every `check-all` run. If a required toolchain component is missing, fail loudly with a clear error message and instructions to install it — do not skip or silently ignore the missing target. Silent skips hide real breakage.
- When loading data from flash (or any other storage) into a local variable, name the variable after the concrete type. Example: `DeviceConfig` data should live in variables like `device_config`, not generic `config` or `flash0`.
- Avoid introducing `unsafe` blocks. If a change truly requires `unsafe`, call it out explicitly and explain the justification so the user can review it carefully.
- Avoid silent clamping; prefer asserts or typed ranges so out-of-range inputs fail fast.
- Prefer `no_run` doctests; use `ignore` only when absolutely necessary (and call out why). Running doctests is best when possible, but rarely feasible for embedded code.
- Always use `rust,no_run` in doctest fences, not just `no_run`.
- For programs that should run forever, use `pending().await` instead of a timer loop.
- **Hide boilerplate in doctests** using the `#` prefix (e.g., `# #![no_std]`). Hide lines that are noise to the reader but required for compilation: `#![no_std]`, `#![no_main]`, and standard imports like `use embassy_executor::Spawner;`. Keep only the essential code showing how to use the API. See the crate-specific sections below for platform-specific imports to hide or show.
- When adding docs for modules or public items, link readers to the primary struct and keep the single compilable example on that struct; other items should point back to it rather than duplicating examples.
- Prefer `const` values defined in the local context (inside the function/example) rather than at module scope when they're only used there.
- Do not add redundant `just` recipes that only mirror an existing `cargo` alias/command. If the behavior is the same, keep only the `cargo` command.
- For `cargo` aliases that target embedded triples, include `--no-default-features` unless there is an explicit, documented reason to keep default features enabled.

## Dependency Upgrade Guardrails

- Treat embedded HAL/driver dependency upgrades as behavior migrations, not just compile fixes. When an upgraded API adds a required parameter, trace the previous behavior/default before choosing a placeholder value.
- Keep dependency-upgrade commits focused when possible. Avoid mixing broad docs/example churn with HAL, network, DMA, executor, or firmware-driver upgrades unless the extra changes are necessary for the migration.
- After upgrading hardware-facing dependencies, run at least one hardware smoke test for each affected subsystem before considering the upgrade complete. For RP WiFi/CYW43 changes, `clock_console` or a minimal WiFi example must get past CYW43 initialization and reach WiFi join or captive portal readiness.

## Generated Files

- Treat generated files under `src/**/_generated.rs` as build outputs, not source of truth.
- When changing generated docs/examples, edit the corresponding generator template in `xtask/src/*_generated.rs` first.
- If you must patch a generated file directly for an urgent fix, make the matching template change in the same PR so regeneration does not revert it.
- Regenerate and verify with `cargo xtask check-docs` (or the crate-level check command) before handing work back.
- When changing generated API surface/docs for macro-backed types, update all four in the same PR: (1) macro source in `src/*.rs`, (2) generator template in `xtask/src/*_generated.rs`, (3) generated stub in `src/**/_generated.rs`, and (4) `xtask/src/main.rs` `check_generated_doc_stubs` expectations.

## Module Structure Convention

This project uses a specific module structure pattern. Do NOT create `mod.rs` files.

- Macros related to a specific submodule should generally live in that submodule (for example, audio macros in `audio_player`) rather than in the top-level module.
- For exported `macro_rules!` macros that conceptually belong to a submodule, keep the user-facing docs/re-export in that submodule and avoid cluttering top-level macro docs. Prefer the existing pattern: `#[doc(hidden)]` on the `#[macro_export]` definition plus an in-module re-export (`pub use macro_name;`) with the full docs on that re-export.
- If you change macro visibility/export style, verify rustdoc placement still matches intent (submodule-focused docs, no unintended top-level macro listing).

Correct pattern:

- `src/foo.rs` or `examples/foo.rs` (main module file)
- `src/foo/bar.rs` (submodule)
- `src/foo/baz.rs` (another submodule)

Incorrect pattern (never use):

- `src/foo/mod.rs` ❌
- `examples/foo/mod.rs` ❌

Example:

```rust
// File: src/wifi_auto.rs (main module)
pub mod fields;
pub mod portal;

// File: src/wifi_auto/fields.rs (submodule)
// File: src/wifi_auto/portal.rs (submodule)
```

## Variable Naming Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CarlKCarlK/device-envoy](https://github.com/CarlKCarlK/device-envoy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
