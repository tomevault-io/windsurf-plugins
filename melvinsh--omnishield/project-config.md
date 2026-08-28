---
trigger: always_on
description: Guidance for coding agents working in this repository.
---

# AGENTS.md

Guidance for coding agents working in this repository.

OmniShield is a non-root Android ad/tracker blocker: a Kotlin/Compose app over a Rust core that
owns the entire packet path behind `VpnService`.

`README.md` is written for someone installing the app, not for working on it. Technical
documentation lives in `docs/`. Some of what follows is repeated there. When they disagree, the
code wins, and fix both.

| File | What to read it for |
|---|---|
| `docs/architecture.md` | The packet path, the filtering layers, routing, and the platform limitations |
| `docs/development.md` | Toolchain, emulators, offline probes, and the version-constraint table |
| `docs/performance.md` | Measured cost, the mechanisms behind it, and the deliberate choices that look like waste. Check there before "fixing" one of them |
| `docs/interface.md` | Material 3 Expressive, and why each screen is shaped the way it is |

The repo is **public** and MIT licensed. `CONTRIBUTING.md`, `SECURITY.md`, `NOTICE.md` and
`CHANGELOG.md` are written for strangers and this file is not, but all of them are readable by
anyone.

## Do not break

- A new dependency in `core/Cargo.toml` must not be GPL. That would conflict with the app's
  licence; `NOTICE.md` explains the check.
- The `## [x.y.z]` headings in `CHANGELOG.md` are parsed by the release workflow. Reshaping
  them silently empties the release notes.
- **`core/Cargo.toml` sets `panic = "abort"`.** A panic cannot usefully unwind across the JNI
  boundary, and the alternative is worse than a crash: unwinding kills only the tunnel thread,
  leaving `VpnService` holding a TUN that nothing services, so every app on the device loses
  connectivity with no error anywhere. Aborting takes the process down and `START_STICKY`
  brings it back. Consequence: no `catch_unwind` anywhere in the core will work.
- cargo-ndk's panic handler dumps the entire process environment to stdout. Do not run it from
  a shell holding live credentials.
- Once `OmniShieldVpnService` calls `ParcelFileDescriptor.detachFd()` and hands the descriptor
  to `nativeStart`, **no packet crosses the JNI boundary**. Preserve this: it is the reason the
  core is in Rust at all.
- `packet::parse` returning `None` means **drop**, never "forward unfiltered": passing a packet
  we failed to parse would be a filtering bypass.

`docs/development.md` has the full version-constraint table. The ones that bite hardest:

- **AGP 9.x is mandatory** (current AndroidX declares `requires Android Gradle plugin 9.1.0 or
  higher`), and AGP 9 has built-in Kotlin, so applying the `kotlin-android` plugin is a hard
  failure.
- **`compileSdk` 37 / `targetSdk` 34** are deliberately different; the platform package is
  `platforms;android-37.0` (the `.0` is required).
- **`minSdk` 29 cannot go lower**: `ConnectivityManager.getConnectionOwnerUid` does not exist
  below it and `/proc/net` scraping was blocked in the same release, so the per-app firewall is
  impossible on anything older.
- material3 is pinned to a **1.5.0-alpha** ahead of the BOM for the Expressive component set;
  the rest of Compose stays stable.

## Couplings that fail silently

None of these produce a compile error when broken. They fail at runtime, usually as "nothing
happens".

1. **JNI symbol mangling.** `core/src/android.rs` exports 14 `Java_io_omnishield_bridge_NativeBridge_*`
   functions matching 14 `external fun`s in `bridge/NativeBridge.kt`. Renaming that object or
   its package requires renaming every Rust export in lockstep. (The package is `bridge`, not
   `native`, which is a Java reserved word.)
2. **Reverse callbacks.** `core/src/jvm.rs` calls Kotlin by name *and JNI signature string*:
   `protect(I)Z`, `lookupUid(ILjava/lang/String;ILjava/lang/String;I)I`, `packageForUid(I)Ljava/lang/String;`
   on `OmniShieldVpnService`. These are `@Keep`-annotated and listed in `proguard-rules.pro`.
3. **The config JSON contract.** `CoreJson.buildConfig` in Kotlin must use the exact serde field
   names in `core/src/config.rs`. A renamed field silently decodes to that field's default.
   `CoreJsonTest` asserts the field names for this reason.
4. **Every off-thread event must wake the tunnel loop.** This one fails as a *hang* rather
   than as nothing happening. The loop sleeps until there is something to do (backstop
   `IDLE_CEILING_MS`, 30 s) instead of waking on a timer, so anything that changes state from
   another thread must call `Runtime::wake`: the JNI config and rule setters, `Runtime::stop`,
   and the DoH worker, whose answers arrive on an mpsc channel the loop only drains while
   awake. `core/src/wake.rs` documents all three. A new producer that forgets to wake looks
   like a hung tunnel, and the 30 s ceiling is the only reason it self-heals instead of
   staying wedged.

## Packet path

Kotlin owns lifecycle, persistence and UI. Rust owns everything on the packet path.

Events flow back by *polling* (`nativeDrainEvents` in `pollCore()`), not by native-to-JVM
callbacks, which would require attaching the tunnel thread to the JVM per event. The poll
interval is **adaptive**, not fixed, and its rules differ by screen state (`PollSchedule`):
while someone is watching (`TunnelRepository.uiActive` *and* the screen interactive), any event

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [melvinsh/omnishield](https://github.com/melvinsh/omnishield) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
