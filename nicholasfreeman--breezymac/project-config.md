---
trigger: always_on
description: Guidance for working in this repository. Read this first.
---

# CLAUDE.md — BreezyMac

Guidance for working in this repository. Read this first.

## What this is

**BreezyMac** is a macOS fan-control application for Apple-Silicon MacBooks. It
is *exclusively* about system temperatures and fan control — no battery %,
memory, or disk UI. It has two processes:

- **App** (`org.WhoCo.BreezyMac`): unprivileged, status-bar–only agent (no Dock
  icon). A status-bar menu switches modes; an on-demand tabbed window configures
  everything. Reads sensors directly (SMC reads need no root).
- **Helper** (`org.WhoCo.BreezyMac.Helper`): a privileged root LaunchDaemon that
  performs SMC *writes* (fan mode/target). Installed via `SMAppService`,
  reachable only over NSXPC.

Only the app touches the UI; only the helper touches fan writes.

## The four operating modes (the central abstraction)

`OperatingMode` (Sources/Shared/OperatingMode.swift):
- **disabled** — app + helper have zero influence; all control returned to macOS.
  This is the safe default. The helper is not even registered until an engaging
  mode first needs it.
- **automatic** — the flagship anti-throttle mode. Fans ramp to hold CPU/GPU
  temps below the throttle threshold: proportional ramp between per-power-source
  target/ceiling setpoints + a dT/dt anticipation term + a **max override** when
  `ProcessInfo.thermalState` is `.serious`/`.critical` (the only public throttle
  proxy on Apple Silicon) or temp ≥ ceiling. See AutomaticController.swift /
  AutomaticConfig.swift.
- **adaptive** — fans follow user-defined curves (per power source).
- **performance** — all fans forced to maximum (curve ignored).

Modes and curves are **power-source aware** (AC vs battery), via IOKit power
sources (PowerSourceMonitor.swift). "Silent" was removed: its battery/quiet
use-case is served by battery-specific setpoints/curves. Automatic and Adaptive
depend on live temps (`OperatingMode.needsTemperature`), so a tick reads them
even when no UI is visible.

**Cool-idle 0-RPM handoff (Automatic + Adaptive).** A forced manual fan target
is firmware-clamped to each fan's minimum (`F{i}Mn`), so BreezyMac cannot itself
command 0 RPM. Instead, when the control fraction sits at/near zero (≤ 0.02) for
`handoffSettleSeconds` (10 s), `FanController` hands *all* fans back to macOS via
`releaseControl()` — which clears `Ftst`, letting macOS's own controller spin
them fully down, to 0. Control is re-taken immediately (no dwell) once the
fraction rises past the re-engage band (≥ 0.08), so a load spike is answered
within one tick; the 0.02↔0.08 gap is the anti-chatter hysteresis. For Automatic
the handoff boundary ≈ the `target` setpoint (below target the proportional term
is already 0); for Adaptive it sits just under each curve's first non-zero knee,
so a curve point of `0%` finally means *off*, not minimum. Because macOS governs
while spun-down, the post-load fan tail follows macOS's fuller thermal model
(chassis/heatsink mass), not our die-only sensors — it can hold minimum airflow
for minutes after die temps drop, then stop. This is intended, not a stall.

## Safety invariants (non-negotiable — these are product requirements)

1. **No influence when the UI isn't active.** The app sends a heartbeat every
   `kHelperHeartbeatInterval` (2 s) while any engaging mode is active. The helper
   runs a **watchdog**: if no heartbeat for `kHelperWatchdogTimeout` (6 s) it
   returns every fan to macOS auto on its own. This single mechanism covers app
   quit, crash, sleep, and lid-close (a slept app can't heartbeat).
2. **Release on sleep / lid close.** The app also proactively releases on
   `NSWorkspace.willSleepNotification` and re-asserts on `didWakeNotification`.
3. **Release on quit.** `applicationWillTerminate` → `FanController.shutdown()`
   releases control; the helper additionally releases on XPC invalidation and on
   SIGTERM/SIGINT.
4. **Disabled means disabled.** In `.disabled` the app releases control, stops
   heartbeating, and remains completely inert. (It need not *unregister* the
   daemon — leaving it registered-but-dormant is acceptable as long as control
   is fully returned to macOS.)
5. **The heartbeat timer MUST run in `.common` run-loop modes.** While the
   status-bar `NSMenu` (or any menu/modal panel) is open, AppKit runs a nested
   run loop in event-tracking mode. A timer registered only in the default mode
   stops firing, the heartbeat lapses, and the watchdog reverts fans mid-use.
   `FanController` adds its tick timer via `RunLoop.main.add(t, forMode: .common)`
   and fires synchronously (`MainActor.assumeIsolated`). Do not regress this.

When touching control flow, do not weaken these. The reference apps we derived
from both *lacked* reset-on-quit/sleep handling — that was their worst bug.

## Idle behavior / polling (keep the app quiet)

Polling is demand-driven so the app idles near 0% CPU:
- The tick timer runs only when `mode.engagesHelper || uiVisible`. In Disabled
  with no menu/window open, the timer is **stopped** entirely.
- A **full** `SMCReader.snapshot()` runs only when UI is visible. When engaged
  but hidden, a tick does just a heartbeat, plus a temperatures-only read for
  Adaptive. `SMCReader` caches resolved temp-sensor keys (no more re-probing the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NicholasFreeman/BreezyMac](https://github.com/NicholasFreeman/BreezyMac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
