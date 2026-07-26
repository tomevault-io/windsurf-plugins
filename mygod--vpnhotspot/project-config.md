---
trigger: always_on
description: This repo is a single-app Android project. Root Gradle files live at the repo top level. The app module is [`mobile`](./mobile), with Kotlin/Java under [`mobile/src/main/java`](./mobile/src/main/java), native code under [`mobile/src/main/cpp`](./mobile/src/main/cpp), Rust daemon code under [`mobile/src/main/rust`](./mobile/src/main/rust), resources under [`mobile/src/main/res`](./mobile/src/main/res), JVM tests under [`mobile/src/test/java`](./mobile/src/test/java), and instrumented tests under 
---

# Repository Guidelines

## Project Structure & Module Organization
This repo is a single-app Android project. Root Gradle files live at the repo top level. The app module is [`mobile`](./mobile), with Kotlin/Java under [`mobile/src/main/java`](./mobile/src/main/java), native code under [`mobile/src/main/cpp`](./mobile/src/main/cpp), Rust daemon code under [`mobile/src/main/rust`](./mobile/src/main/rust), resources under [`mobile/src/main/res`](./mobile/src/main/res), JVM tests under [`mobile/src/test/java`](./mobile/src/test/java), and instrumented tests under [`mobile/src/androidTest/java`](./mobile/src/androidTest/java). Product-specific source lives in [`mobile/src/google`](./mobile/src/google) and [`mobile/src/freedom`](./mobile/src/freedom).

## Build, Test, and Development Commands
Use Gradle from the repo root.

- `./gradlew :mobile:compileDebugKotlin`: fast Kotlin compile check.
- `./gradlew :mobile:testDebugUnitTest`: run JVM/unit tests.
- `./gradlew :mobile:installDebug`: install the debug build on a connected device.
- `./gradlew :mobile:connectedDebugAndroidTest`: run instrumented tests on device/emulator.

## Coding Style & Naming Conventions
Follow existing Kotlin style: 4-space indentation, concise expression bodies only when clear, and existing naming patterns. Match nearby code before introducing new structure.

- Do not add single-use helpers, wrappers, compat objects, or throwaway one-off classes/data classes unless there is real reuse or a correctness reason.
- Do not break logic into multiple one-off private functions just to "organize" code. If a private function is only called once and does not create real reuse, keep that logic inline in the real entry point.
- Be especially strict about this everywhere. Do not hide one linear control flow behind a pile of `start*`/`stop*`/`update*`/`refresh*` helpers unless each helper has clear reuse or isolates genuinely tricky logic.
- Do not introduce single-use temporary variables that only rename a value for the next line or two. Inline them unless they prevent duplicated work or make a genuinely complex expression clearer.
- Do not keep parallel shadow state in multiple vars when an existing data class or state object can be the single source of truth.
- Do not add defensive `toList()`/`toSet()`/similar copies without a concrete ownership or mutation reason. Keep the copy only when it changes representation, breaks a live view, or protects iteration from mutation.
- Do not use `runCatching` for new code; follow the repo’s normal explicit `try`/`catch` style.
- Do not suppress unexpected exceptions. Best-effort cleanup should catch only the expected failure mode and rethrow the rest.
- Preserve existing comments; do not casually shorten or rewrite them.

## Kotlin Concurrency Design
Prefer resource-owner concurrency over broad locks or global serialization.

- For UI-backed state and lightweight suspending operations, prefer a Main-confined owner using `Dispatchers.Main.immediate`, with explicit in-flight and pending state when operations must run to completion.
- For ordered command or state transitions, prefer a single owner worker, channel, or pending-state loop over launching independent jobs that can interleave.
- Use `Dispatchers.Default.limitedParallelism(1, "...")` for non-UI owner-local mutable state confinement when multiple coroutine entry points need a shared lane, but do not rely on it for run-to-completion ordering across suspensions.
- Use `Mutex` for narrow, local critical sections where the protected invariant is clear. Do not use a daemon/global mutex to hide caller-owned lifecycle races.
- Do not run blocking work on Main. Main-confined owners may call suspending/nonblocking APIs, but blocking I/O, sleeps, or CPU-heavy work must stay off Main.

## Rust Daemon Code Hygiene
Rust daemon code should be event-driven and async-first. Prefer Tokio readiness, cancellation tokens, notifications, channels, and deadline timers over polling loops, fixed sleeps, or manually managed worker threads.

- Keep [`docs/vpnhotspotd`](./docs/vpnhotspotd) in sync with daemon internals. When changing
  `mobile/src/main/rust/vpnhotspotd`, `mobile/src/main/proto/daemon.proto`, or Kotlin daemon
  control code under `mobile/src/main/java/be/mygod/vpnhotspot/root/daemon`, update the relevant
  daemon docs if ownership, lifecycle, cleanup, NAT66, DNS, routing, neighbour monitoring, or
  structured error semantics change. If no daemon doc update is needed, state why in the change
  summary.
- Do not let daemon docs gloss over external side effects. If a Rust daemon change adds, removes, or
  changes kernel, netfilter, netd, resolver, socket, file descriptor, process, or Android system
  state, document the trigger, exact external state or command shape, rollback/stop behavior, Clean

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mygod/VPNHotspot](https://github.com/Mygod/VPNHotspot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
