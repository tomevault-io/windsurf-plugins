---
trigger: always_on
description: This is the canonical instructions file for any AI coding agent (Claude Code,
---

# AGENTS.md — MOTO-CTRL agent instructions

This is the canonical instructions file for any AI coding agent (Claude Code,
other AI tools, etc.) working in this repository. It exists so that any agent
picking up this project — in this session or a future one — can work safely from
this file alone, without needing the original project-kickoff conversation.

`CLAUDE.md` at the repo root is just a pointer to this file. If you are an agent
and you only read one file before starting work, read this one.

## What MOTO-CTRL is

MOTO-CTRL is an open-source programmable motorcycle control board (an alternative
to the Motogadget m.Unit Blue): a 12-output, 8-input ESP32-S3 board that switches
motorcycle electrical loads (lights, signals, ignition, starter, horn, aux) over
BLE, with a phone-as-key immobilizer, a companion mobile app, and a hardware
design that's sold assembled by the project — Speeduino-style: open design,
commercial boards.

## Non-negotiable safety requirements

These override all feature requests, refactors, "quick fixes," and stylistic
preferences. If a change would violate one of these, stop and raise it instead of
implementing it. These are recorded verbatim from the project's founding
specification and must not be edited, softened, or reinterpreted away:

1. **Ride-safe failure.** BLE disconnect, app crash, or phone absence must NEVER
   change output state. If the MCU reboots (watchdog or brownout), outputs must be
   restored from persisted state in <250ms. Headlight, ignition, and brake-light
   channels may never be dropped by any software error path while the bike is in
   the "running" state.
2. **Immobilizer only engages when stopped/off.** Lock state may only be entered
   from the "parked" state — never while ignition output is live.
3. **Layered unlock — never lock the rider out.** All configured unlock methods
   work simultaneously: (a) phone-as-key BLE proximity/tap, (b) button cheat-code
   on the handlebar buttons, (c) traditional ignition-switch input mode.
   **At least one non-phone method must be configured before the immobilizer
   can be enabled** — either the button code or the ignition-switch input — so a
   dead phone can never strand the rider. Physical factory reset: within 5s of
   power-on, press BOOT and hold it for 10s → wipes bonds + config after a
   distinct LED pattern confirmation.

   *Corrected 2026-08-02.* This previously read "hold BOOT during power-on for
   10s", which is not physically achievable: BOOT is GPIO0, and holding it
   through reset is the ESP32-S3's strapping for UART download mode, so the
   firmware never runs. The security properties are unchanged — physical
   access, a deliberate 10s hold, a distinct confirmation — only the gesture
   is now one a rider can actually perform. See `firmware/main/factory_reset.h`
   for why the window is watched on the app tick rather than blocking boot.

   *Amended 2026-08-01, by the project owner.* This requirement previously read
   "the button code is always available as fallback even when phone-as-key is
   enabled", i.e. the cheat-code was mandatory. A rider with an OEM ignition key
   switch wired to an input already has a physical, non-phone way in, and
   forcing them to also set a button code they will never use added no safety.
   The invariant that matters — never only the phone — is unchanged and is
   enforced in `mc_lock_config_validate()`.
4. **Phone-as-key must be cryptographically sound.** BLE bonding with LE Secure
   Connections + application-layer challenge-response (device-stored key signs a
   per-session nonce). MAC address alone is never trusted. Support multiple
   paired phones, key revocation, and an ownership-transfer flow (full wipe +
   re-pair).
5. **Brake light priority.** Any brake-flasher pattern must guarantee the brake
   light is ON whenever the brake input is asserted; flash pattern is an
   attention prefix, configurable, defaulting to a short 3-pulse burst then
   solid. Include a note in the app that flash patterns are not legal in all
   jurisdictions, default OFF.
6. **Starter protection.** Starter output is not triggerable from the app
   (hardware button path only), is inhibited while the engine-running state
   is detected — an unconditional check that an assigned ignition channel
   (if any) is live, plus an optional, off-by-default voltage-based
   detection layer (charging voltage > threshold) — and supports an optional
   neutral/clutch interlock input assignment.

   *Amended 2026-08-17, by the project owner.* This previously read
   "detected (voltage-based detection: charging voltage > threshold)" as the
   sole, always-on mechanism. Two problems surfaced on the bench: voltage
   alone cannot distinguish a running alternator from a booster pack or a
   bench PSU holding the line above the threshold, so a rider jump-starting
   a dead bike found the starter refused exactly when they needed it most;
   and nothing forced `engine_running` back to false when the ignition was
   switched off, so a booster left connected above threshold with the key
   off could report "running" indefinitely. Voltage-based detection is now
   an explicit opt-in — `engine_run_voltage_detection_enabled`, OFF by

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [silasboon/moto-ctrl](https://github.com/silasboon/moto-ctrl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
