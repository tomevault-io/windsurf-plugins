---
trigger: always_on
description: This file provides project instructions for AI coding agents working on Napaxi.
---

# AGENTS.md

This file provides project instructions for AI coding agents working on Napaxi.

## Project Shape

- Napaxi is a mobile-native SDK project, not a Flutter-only project.
- `crates/core/` contains the Rust runtime kernel and adapter-facing API shared by the SDK.
- `crates/features/` contains feature-domain crates used by core, not adapter entrypoints.
- `vendor/` contains third-party patched or vendored dependencies such as `libsql-patched`.
- `crates/core/src/api/` is the common Napaxi Core API boundary for all adapters.
- `packages/` contains SDK adapter packages and binding bridge packages.
- `packages/api_bridge/` is the Rust FFI/FRB bridge over `napaxi_core::api`.
- `packages/api_contract/` is the adapter-layer API contract (methods, errors, capability matrix, fixtures) used by parity and integration checks.
- `packages/flutter/` is the Flutter adapter package.
- `packages/android/` is the native Android Kotlin SDK adapter.
- `packages/ios/` is the native iOS Swift Package SDK adapter.
- `packages/agent_provider/` is the provider-side SDK for Agent App actions (separate Android/iOS packages).
- Do not create a sibling `sdk/` tree or reintroduce `packages/napaxi_sdk`.
- `examples/` contains demo apps. Demo apps must consume SDK adapters and must not own reusable SDK implementation logic.
- `examples/flutter/` is the first demo target and depends on `../../packages/flutter`.
- `examples/flutter/` is an integration sample and capability validation app,
  not a second SDK layer. Keep UI state, demo-only models, pages, panels, and
  mockable demo clients there, but move reusable SDK/runtime behavior to
  `crates/` or `packages/`.
- `docs/` contains architecture, integration, and contribution documentation.
- `tools/scripts/` contains shared build, codegen, hygiene, and packaging helpers.

## SDK Boundary

- Runtime behavior belongs in `crates/`, especially `crates/core`.
- Packages must not depend on `crates/features/*` directly; feature behavior reaches adapters through `napaxi_core::api`.
- Feature crates must not depend on `crates/core`.
- Do not place third-party patched/vendor crates under `crates/core/` or `crates/features/`; use `vendor/`.
- SDK adapters must enter Rust core through `napaxi_core::api`; do not call `mobile_*` implementation modules directly from adapter code.
- Packages must not call top-level core internals such as `napaxi_core::android_assets`, `napaxi_core::android_linux_env`, or `napaxi_core::ios_ish_env`; expose adapter-needed hooks through `napaxi_core::api`.
- `mobile_*` module names are legacy implementation names, not public API. Do not add new `mobile_*` modules or crate-internal `crate::mobile_*` references. New runtime behavior should be typed/internal first, then explicitly exported through `api`.
- Do not add broad `pub use crate::mobile_*::*` exports in `crates/core/src/api`; keep API exports as small whitelists.
- Cross-cutting extension points belong in the core capability registry under
  `crates/core/src/capabilities/` and must be exposed through
  `napaxi_core::api::capability`.
- New LLM providers, built-in tools, MCP surfaces, platform tools, policy
  hooks, and background services must define a capability contract before they
  are exposed to adapters or demo apps.
- Channel/provider ownership has four layers. `crates/core` owns the reusable
  runtime, provider contract, capability/policy gates, and official first-party
  sans-IO protocol kits. `packages/api_bridge` only exposes `napaxi_core::api` to
  adapters. SDK adapter packages such as Flutter/Android/iOS stay thin: host
  context, lifecycle/background/permission glue, provider host wrappers, and FFI
  calls. Provider implementations own real platform I/O and may live in host
  apps, external packages, or optional first-party provider modules; demos must
  not own reusable provider/runtime logic.
- Official channel protocol decisions that must stay cross-adapter consistent
  should be core-owned as sans-IO helpers exposed through `napaxi_core::api`.
  Examples include payload shaping, endpoint routing, inbound normalization,
  webhook/gateway state machines, signing checks, Markdown/fallback mapping, and
  error/retry classification. Pin these protocol kits with shared fixtures so
  Flutter, Android, iOS, and tests bind one source of truth.
- External developers must be able to add channels without modifying `crates`.
  Their extension path is the SDK provider contract: register a manifest,
  submit normalized inbound messages, lease/deliver outbound messages, and let
  core handle routing, sessions, history, ask-human, policy, and outbound queue
  state. Add core APIs only when the shared channel contract is insufficient,
  not for every third-party provider.
- Real provider transports stay outside `crates/core` when they depend on
  platform lifecycle, OS permissions, vendor SDKs, secure storage, background
  execution, long-lived sockets, heartbeat timers, QR/login UI, Bluetooth, or
  host network policy. Core may own stateless, platform-neutral transports for
  domains that already live in core, but channel providers should default to
  sans-IO protocol in core plus transport in provider/host code.
- Capability IDs use stable reverse-domain-style SDK names such as

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antgroup/Napaxi](https://github.com/antgroup/Napaxi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
