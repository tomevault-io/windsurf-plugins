---
trigger: always_on
description: > ↑ [data/device/CLAUDE.md](../CLAUDE.md) · [apps/mewbo_aura/CLAUDE.md](../../../../../../../../../../CLAUDE.md) · [root](../../../../../../../../../../../../CLAUDE.md)
---

> ↑ [data/device/CLAUDE.md](../CLAUDE.md) · [apps/mewbo_aura/CLAUDE.md](../../../../../../../../../../CLAUDE.md) · [root](../../../../../../../../../../../../CLAUDE.md)

# Screen control at shell UID — data/device/shizuku/

Scope: the Shizuku-backed half of device control. `ShizukuDeviceControl` (the app-side binding),
`DeviceUserService` (the shell-UID implementation), and the pure logic each is built from:
`ElementPruner`, `DisplayGeometry`, `SettlePolicy`, `UiSnapshot`, `ScreenCapture`.

## Two processes, and only one of them is the app

`DeviceUserService` does **not run in the app's process**. Shizuku starts it as a separate
`app_process` at uid 2000 and hands the app a binder. Nothing in it may touch app state, Hilt, or
any singleton — only what arrives over the binder. `ShizukuDeviceControl` is the app-side half and
the only thing that binds.

**The binding is held, not per-action.** That is the whole latency argument: a spawn per tap would
pay process startup every time. Measured on the dev container, the marginal cost of an
`app_process` spawn is **~16 ms** — two orders of magnitude below the ~1s figure the design was
originally argued from, so the *reason* to hold the binding is that `newProcess` is deprecated and a
held channel is the right shape, not that a spawn is ruinous. **The real cost centre is the element
read at ~2.0s**, which is ~124x a tap. Optimise there or nowhere.

## Index addressing is a correctness mechanism, not an ergonomic one

The model names an element index; the client resolves it to a centre point. There is deliberately
no coordinate argument anywhere in the tool surface. This removes by construction the failure where
coordinates computed against a downscaled screenshot are applied to full-resolution device space:
every tap lands proportionally wrong, hitting a real control rather than erroring, so nothing
reports a problem.

Two consequences:

- **Geometry never crosses the wire.** Four coordinates per element would be paid for on every
  observation and read by nobody. Pinned by a test, because it is the kind of field a future edit
  adds back "for completeness".
- **A stale index must be a structured error, never a silent mis-tap.** The list the model holds may
  be several turns old. `DeviceActionHandler.resolve` re-reads at action time and refuses with the
  actual element count, which is a state the model can recover from.

## An injected tap hits the TOPMOST window — including one of ours

`input tap` is delivered to whatever window is on top at those coordinates, so any overlay this app
draws intercepts the taps it is trying to inject. The device-control Stop pill sits bottom-centre
and would have ended the grant it was acting under. The cure is temporal — `ScreenCaptureVeil` wraps
`tap`/`swipe`/`type` (`type` included: it taps to focus the field first), and a window that is not
on screen cannot be hit. See [`ui/control/CLAUDE.md`](../../../ui/control/CLAUDE.md) for the window
side, and [`data/device/CLAUDE.md`](../CLAUDE.md) § "The capture veil" for the seam.

Two related window facts, both of which fail with NO error, if you ever add another overlay:
`TYPE_APPLICATION_OVERLAY` above **0.8** obscuring alpha silently drops every touch to the app
underneath (Android 12 untrusted-window rule), and a view added straight to the `WindowManager` does
not inherit the manifest's hardware acceleration — AGSL has no software path, so a shader draws
nothing without `FLAG_HARDWARE_ACCELERATED`.

## `show_touches` cannot verify an injected tap — do not reach for it

Injected events enter the pipeline at `InputDispatcher`, **downstream of
`PointerChoreographer`**, which is the only stage that draws touch spots. So the setting renders
nothing for our taps while still rendering the human's — a signal that is silent when correct and
misleading when wrong. Verification is by EFFECT (did the screen change), which is what
`SettlePolicy` is for.

## The settle is bounded, and the bound is not the interval

Three identical reads, 0.5s apart, 6.0s cap. **Never an unbounded idle wait** — `uiautomator2`
disables framework idle-waiting by default because a device that never idles (an animation, a
video, an ad) hangs the caller, and inside a 30s dispatch budget that is a timeout generator.

The real worst case is the cap **plus one read**, since the deadline is checked before starting a
poll rather than mid-read. With a ~2.0s read that is ~7s, not ~1s. Still inside the budget, still
incapable of running away.

## `uiautomator dump` ships, against the original plan, on a measurement

It was rejected as "an `app_process` launch — same cost class as the rejected route". Measured, the
spawn is ~16 ms and the other ~2003 ms is the accessibility-tree read itself, which any method pays.
Reading the tree in-process saves the 16 ms, not the 2 seconds.

What it does avoid is the file round-trip: the dump goes to stdout and is parsed in memory, so there
is no `/sdcard` write and no temp file. `UiSnapshot` isolates it behind one function precisely so a
held `UiAutomation` connection can replace it later without touching the pruning or the wire format.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bearlike/Assistant](https://github.com/bearlike/Assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
