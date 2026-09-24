---
trigger: always_on
description: ESP32-S3 (XIAO) BLE sensor pucks for an e-bike: BNO055 IMU + BMP280 barometer
---

# Tripper-DL1 — notes for Claude

ESP32-S3 (XIAO) BLE sensor pucks for an e-bike: BNO055 IMU + BMP280 barometer
+ Talaria CAN tap, streaming 5 Hz telemetry to the Tripper iOS app (sibling
repo `../Tripper`). Two builds, wire-compatible on purpose: **Full**
(`tripper_puck.ino` — GPS, OLED, buttons) and **Light** (`tripper_light.ino` —
IMU + baro + CAN only). The rider currently runs Light.

**Keep the two `.ino` files in step** — packet structs, CAN decode, UUIDs and
the OTA block are duplicated by design, and every change lands in both.

## The verdict that shapes everything (settled 2026-08-08 — do not relitigate)

The BNO055's fusion **invents attitude its own sensors contradict**. Proven by
the v0x04 raw-accel channel on `Tripper_20260807_232541.trip`: on 118 s of
straight, steady riding the raw accelerometer's implied roll was −0.19° ± 1.04°
while the fusion's roll wandered ±13.6° (to −24° at 69 km/h), correlation
+0.018 between them. Corner sign inverted in 14/15 corners. Mount, axes, gyro,
barometer, I2C (flat quatRejects) all cleared. Full history and the ruled-out
table: `docs/lean-investigation.md`; what a correct instrument would look like:
`docs/direct-attitude-sensing.md`.

Consequences baked into the code:

- The quaternion/linear-accel channel is **diagnostic only** — kept in the
  packet deliberately (a wrong channel you record is a diagnosable channel).
  The app computes lean and slope itself from gyro + CAN speed + baro.
- **No sensor purchase fixes lean** — in a balanced corner the net force runs
  through the bike, so any gravity-referenced device reads upright (measured:
  lateral g 0.015 while moving). Lean = `atan(v·ψ̇/g)` + gyro, period.
- **Slope's distance axis must be GPS-scaled, never raw wheel** — the driven
  wheel over-read 152% on a climb at 60% slip (`docs/lean-investigation.md`,
  "Slope's distance").

## Sign conventions (empirically anchored — see the iOS repo's CLAUDE.md too)

Sensor frame: X forward, Y rider's left, Z up. So a right-hand turn is
**negative** gyro Z (right-hand rule), and the app's right-positive lean scale
means its kinematic term negates gyro.z. Right roll rate = +gyro X. Raw-accel
roll, right-positive, = `atan2(+accY, +accZ)`; the bike parked on its
(left-side) kickstand reads ≈ −9.5° and anchors the sign.

## Packet protocol — the additive rule

Never reorder or resize existing fields. New fields append; the app gates on
**length**, not version, so old apps parse new packets and drop the tail.
Telemetry is 92 B, `ver 0x07`: the 0x06 layout plus `canOdo_km` on the tail
(the bike's odometer, whole km, from CAN `0x402[2:4]` — see "The odometer"
below). 0x06 was 0x05 plus `accDev_mg`.
0x05 was byte-identical to 0x04 with a semantics change — the gyro/raw-accel
fields carry the **mean over the 200 ms packet window** (~20 samples at
100 Hz) instead of the newest sample, and the baro is read at 5 Hz (was 1 Hz —
it staircased under the app's 10 m slope fit). Status is 15 B:
14 + appended `otaState` (0 off · 1 AP up · 1+n = n WiFi clients).

Control writes: `0x01` marker · `0x02` mount zero (refused until accel cal
usable) · `0x03` identify · `0x04` ride state · `0x05` + [on u8] WiFi
flashing mode · `0x06` + [on u8] raw stream (boots **on**) · `0x07` + [mode u8]
ride-mode override (0 release · 1 Eco · 2 Sport) · `0x08` + [level u8] regen
override (0 release · 1–4). Status is 16 B since 0x07: `ovrState` on the tail,
bits 3:0 mode · bits 7:4 regen, reporting what the puck IS holding rather than
what was asked for.

### `accDev_mg` — the peak beside the mean (0x06, 2026-08-09)

Largest `|‖a‖ − 1 g|` seen at 100 Hz inside the packet's window, in mg.

It exists because **the app had a gate it could not trip**. The estimator only
takes its "down" reference from the accelerometer while `‖a‖` is near 1 g, but
it was testing the window MEAN — the one number structurally incapable of
showing a jolt inside its own window. The 100 Hz corpus settled it: real `‖a‖`
was outside 0.8–1.2 g on **17–20% of moving samples**, the means on **0.6–1.3%**.
The gate had been tuned against a signal already smoothed into compliance.

The general lesson, worth more than the field: *a summary statistic can hide
exactly the event a gate on that statistic is meant to catch.* When the app
needs to know something about the window rather than its average, compute it
on the puck — a mean cannot be un-averaged downstream.

Not `maxG_mg`, which is the peak of the fusion's LINEAR accel and therefore a
product of the fusion this channel exists to cross-check.

## The puck now transmits — bike control (0x07, 2026-08-10)

`0x490` is a **command from the dash to the motor controller**, not the echo it
was documented as for months. Two bench measurements settled it: on every mode
button press `0x490` moved 20–27 ms *before* `0x202`, and injecting `0x490`
with Eco while the dash went on sending Sport drove the controller's demand
floor 1100 → 750. So the bike obeys whoever spoke last on that ID, and
`TWAI_MODE_LISTEN_ONLY` became `TWAI_MODE_NORMAL` in both builds.

**"It never transmits, so it cannot disturb the bike" is no longer true.** What
replaces it: nothing is sent unless the rider asks, and every failure path

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kbirand/Tripper-DL1](https://github.com/kbirand/Tripper-DL1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
