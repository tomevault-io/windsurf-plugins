---
trigger: always_on
description: Context file for Claude Code. Read this before changing anything.
---

# Density Toggle - Wear OS display density control

Context file for Claude Code. Read this before changing anything.

## Goal

A Galaxy Watch Ultra (480x480 round, Wear OS / One UI Watch) runs sideloaded
**phone** APKs - YouTube, Uber, Bitwarden, Proton - whose layouts are built for
a large rectangular screen. At stock density their UI overflows the round
display and controls fall outside the visible circle.

The app lowers system display density so those layouts fit, and puts it back
afterwards. Auto-first since v5.9:

- **Auto** - an accessibility service that shrinks when a chosen app comes to
  the foreground and restores when it leaves. Each chosen app can have its
  own density (default from the main screen, per-app override in the picker)
  and its own full-size-typing setting (per-app chip in the picker; no global
  toggle anymore).
- **"Full-size typing" was removed in v5.15** after empirical retesting: the
  current One UI Watch keyboard works fine at low densities (its
  numbers/symbols gesture functions at 140), so lifting density while the
  keyboard was up no longer bought anything and cost a chip + a state
  machine. If keyboard breakage at low dpi ever returns, the git history has
  the whole implementation (kbPause/kbPaused/kbLiftFor + KB_SAFE_DPI=190) -
  re-test on the device before resurrecting it.
- **Reach** (v5.12, optional, off by default): corner controls of phone
  layouts are physically outside the round panel at every density - overscan
  was removed in Android 11 and One UI Watch's WM coerces freeform back to
  fullscreen (both verified on-device), so proxy-clicking is the ONLY
  workaround. The service shows a TYPE_ACCESSIBILITY_OVERLAY chip while a
  watched app is shrunk; the panel lists clickable nodes whose centre lies
  outside the inscribed circle and fires ACTION_CLICK on the chosen one.
  Overlay sizes are PHYSICAL px, never dp - dp shrinks with the density
  override, exactly when the chip is needed. Useless for unreadable trees
  (YouTube, invariant 10).
- The manual shrink workflow was removed in v5.9. The only manual control is
  a **Restore to stock** pill that appears while the screen is actually
  shrunk - the on-watch escape hatch. Do not remove it: without it a stuck
  shrink needs adb. (v6.0 briefly made it always-visible as "Reset to
  default"; reverted - it caused layout churn.)
- **Rotation is NOT an in-app feature** (v5.10 tried, v5.10.1 removed it -
  learn from this): shrinking density makes phone apps use tablet layouts,
  which honour SENSOR orientation, so apps rotate with the wrist. The
  `accelerometer_rotation` setting cannot stop sensor-orientation requests,
  so an in-app toggle built on it LOOKS like it works but doesn't. The mode
  that works - `cmd window fixed-to-user-rotation enabled` - is gated by a
  signature permission the app can never hold, is adb-only, and is persisted
  by the system across reboots. It lives in install.sh (`--lock-rotation`)
  and INSTALL.md. Do not re-add a rotation toggle to the app.

**Hard requirement from the user: the watch must not depend on a phone or
computer during normal use.** A one-time `adb` install plus permission grant is
acceptable (the grant persists across reboots). Anything needing a per-boot
command from another device is not.

## Current status

| Area | State |
|---|---|
| Builds and dexes | **Working** on macOS with build-tools 35.0.0 |
| APK installs on watch | Not yet confirmed |
| Density actually changes from the app | **Not yet confirmed** |
| Auto mode / accessibility service | **Not yet confirmed** |
| `adb shell wm density 240` on this watch | **Confirmed working** |

The last item matters: the underlying mechanism is known good on this exact
device. What is unproven is whether the *app* can reach it.

**First thing to check on device:** open the app and read the small grey line
at the bottom of the main screen. It reports which route worked or the exact
exception. Do not guess from behaviour - that line is the diagnosis.

| Grey line | Meaning |
|---|---|
| `via binder/direct` | working, plain reflection |
| `via binder/exempt` | working, needed the hidden-API exemption |
| `via exec wm` | working via the `wm` command |
| `SecurityException: Must hold permission...` | grant missing - re-run `install.sh` |
| `NoSuchMethodException` | hidden-API filtering blocked reflection; next step is a proper bypass (LSPosed HiddenApiBypass) |
| `accessibility: ...` | self-enabling the service was refused; use the settings fallback |

## How it works

`WindowManagerService.setForcedDisplayDensityForUser()` is guarded by exactly
one thing:

```java
if (checkCallingOrSelfPermission(WRITE_SECURE_SETTINGS) != GRANTED)
    throw new SecurityException(...);
```

`adb shell wm density` works because the **shell uid holds that permission** -
not because the caller is a shell. Any app holding it may make the same call.
So the app is granted `WRITE_SECURE_SETTINGS` once via `pm grant` and then
makes the binder call directly through reflection.

`IWindowManager` is hidden API, hence:

- `targetSdkVersion` is pinned to **27** on purpose. Hidden members marked
  `max-target-o` stay reachable for apps targeting API 27 or lower. **Raising
  targetSdk will probably break density changing.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [channelramble/DensityToggle](https://github.com/channelramble/DensityToggle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
