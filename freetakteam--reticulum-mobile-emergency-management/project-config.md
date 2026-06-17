---
trigger: always_on
description: Guidance for coding agents working in `reticulum_mobile_emergency_management`.
---

# AGENTS.md

Guidance for coding agents working in `reticulum_mobile_emergency_management`.

## Project Snapshot

- This repository is a mixed workspace:
  - `apps/mobile`: Vue 3 + Vite + Capacitor mobile/web client
  - `packages/node-client`: TypeScript bridge used by the app to talk to the native plugin surface
  - `crates/reticulum_mobile`: Rust runtime, UniFFI bridge, and LXMF/Reticulum integration
  - `tools/codegen`: UniFFI binding generation scripts
  - `e2e`: Playwright end-to-end coverage
- Primary product focus is emergency coordination over Reticulum mesh networking, including peer discovery, action messages, event replication, and telemetry.

## Working Rules

- Start from the repo root unless a package-specific command clearly belongs elsewhere.
- Check `git status` before editing. This repo often has generated Android/Rust artifacts in the worktree.
- Do not hand-edit generated or build output unless the task is explicitly about generated artifacts or native packaging.
- Keep fixes scoped. A UI change should not casually rewrite transport or runtime behavior.
- Prefer updating the real source of truth rather than patching copied artifacts.
- Use the compiled `LXMF-rs` implementation through the existing Rust bridge and generated bindings. Do not recreate LXMF protocol functionality in TypeScript, Vue stores, or ad hoc Rust compatibility code when the compiled library already provides it.

## High-Value Directories

- `apps/mobile/src/views`: route-level screens
- `apps/mobile/src/components`: reusable UI pieces
- `apps/mobile/src/stores`: Pinia stores; most app behavior lives here
- `apps/mobile/src/utils`: protocol helpers, peer parsing, replication helpers, mission sync helpers
- `apps/mobile/src/services`: platform-facing helpers such as sharing, notifications, telemetry helpers
- `apps/mobile/src/types/domain.ts`: shared app domain types
- `packages/node-client/src/index.ts`: TS client boundary for the native bridge
- `crates/reticulum_mobile/src/runtime.rs`: main Rust runtime behavior
- `crates/reticulum_mobile/src/sdk_bridge.rs`: SDK-facing LXMF bridge layer
- `crates/reticulum_mobile/src/jni_bridge.rs`: native boundary used by the mobile side
- `crates/reticulum_mobile/src/reticulum_mobile.udl`: UniFFI interface definition
- `docs/architecture.md`: transport and replication architecture notes

## Generated And Volatile Paths

Treat these as generated or disposable unless the task explicitly targets them:

- `node_modules/`
- `target/`
- `playwright-report/`
- `test-results/`
- `tmp/`
- `apps/tmp-playwright-ui.err`
- `apps/mobile/tmp-playwright-ui.err`
- `apps/mobile/android/app/build/`
- `apps/mobile/android/app/src/main/jniLibs/`
- `apps/mobile/android/uniffi/`
- `apps/mobile/ios/uniffi/`

On Windows, broad recursive directory scans can fail inside Android build intermediates. Prefer scoped searches over targeted source directories instead of walking the entire repo.

## Expected App Conventions

- Vue code is written with Vue 3 Composition API and `<script setup lang="ts">`.
- TypeScript is `strict` in both the app and `packages/node-client`.
- Pinia stores hold most stateful behavior. Keep business logic in stores and utilities, not inside view templates.
- Reuse existing domain types from `apps/mobile/src/types/domain.ts` before inventing near-duplicates.
- Keep wire/protocol helpers centralized in `apps/mobile/src/utils` and Rust runtime files rather than scattering message-shape logic across components.
- App-wide button press feedback is defined on global `button` rules in `apps/mobile/src/styles.css`; component buttons should set the existing CSS custom properties rather than adding one-off `:active` behavior.
- Maintain the existing style conventions in touched files:
  - double quotes
  - semicolons
  - explicit typing when it improves clarity at boundaries

## Rust Skills Integration

Apply these additional rules whenever a task touches Rust code, `Cargo.toml`, or the UniFFI/native bridge:

- Treat the installed Rust skills bundle as the default routing layer for Rust work:
  - general Rust questions or ambiguous Rust tasks: `rust-router`
  - ownership, borrowing, lifetimes, and move errors: `m01-ownership`
  - smart pointers and resource ownership patterns: `m02-resource`
  - error modeling and propagation: `m06-error-handling`
  - async, `Send`/`Sync`, threading, and channels: `m07-concurrency`
  - `unsafe`, FFI, raw pointers, JNI, and bridge boundary reviews: `unsafe-checker`
- For new Rust crates or new `Cargo.toml` package sections created in this repo, default to:
  - `edition = "2024"`
  - `rust-version = "1.85"`
  - `[lints.rust] unsafe_code = "warn"`
  - `[lints.clippy] all = "warn"` and `pedantic = "warn"`
- Prefer domain-correct design fixes over borrow-checker workarounds. Do not reach for cloning or ownership duplication until the ownership model is justified by the runtime and protocol design.
- Use `?` and typed error propagation in library/runtime code instead of `unwrap()` or `expect()`, unless a crash is intentionally part of the boundary behavior.
- Every `unsafe` block must carry a nearby `// SAFETY:` comment that states the invariant making the block sound.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FreeTAKTeam/reticulum_mobile_emergency_management](https://github.com/FreeTAKTeam/reticulum_mobile_emergency_management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
