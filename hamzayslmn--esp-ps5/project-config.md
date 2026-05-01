---
trigger: always_on
description: An Arduino / ESP-IDF library that lets an ESP32 act as a Bluetooth Classic
---

# esp-ps5 — How the system works

An Arduino / ESP-IDF library that lets an ESP32 act as a Bluetooth Classic
host for a PS5 DualSense controller. The ESP32 pretends to be a PS5
console, the controller pairs with it, and the library exposes
buttons / sticks / triggers / status to the user sketch and accepts
LED / rumble / player-LED / mute-LED / adaptive-trigger output back to
the controller.

The protocol layer is a verified port of Sony's DualSense BT report
format (cross-checked against Linux's `drivers/hid/hid-playstation.c`).

## File layout

```
   Sketch (.ino)
        |
        v
   ps5Controller.{h,cpp}      Arduino class: scan/pair/auto-reconnect,
                              fluent output API, user callbacks. Owns the
                              global `ps5` instance with all input as flat
                              public fields.
        |
        v
   ps5_bytes.cpp              ALL byte/bit work. Builds the wire-correct
                              BT 0x31 OUTPUT report (rumble, lightbar, LEDs,
                              adaptive triggers) with trailing CRC32, and
                              parses the BT 0x31 INPUT report straight into
                              the global `ps5` flat fields.
        |
        v
   bluedroid/bluedroid.cpp    L2CAP transport (HID PSMs 0x11 + 0x13) +
                              GAP/SPP bring-up. Vendored Bluedroid plumbing.
   bluedroid/                 Vendored ESP-IDF Bluedroid internal headers.
                              (Implementations live inside the Arduino-ESP32
                              core's Bluedroid blob; only the headers are
                              vendored because ESP-IDF doesn't expose them.)
```

## How a connection happens

1. Sketch calls `ps5.begin()` (or `ps5.begin(secs)` / `ps5.begin("AA:BB:..")`).
2. `begin()` brings up the BT controller, Bluedroid, GAP, SPP, and registers
   HID **control** + **interrupt** PSMs (0x11 + 0x13) as L2CAP listeners.
3. With no MAC stored yet, `begin()` runs an auto-pair scan: every unique
   nearby device is added to a **growing linked list** (one node per MAC,
   ~10 B). Scan early-exits the moment a device named "DualSense" or
   "Wireless Controller" appears. Its MAC is fed to `ps5_l2cap_connect()`,
   which fires an outbound `L2CA_CONNECT_REQ` on PSM 0x11.
4. Both L2CAP channels configure successfully → on the up-edge, bluedroid
   calls `ps5_scan_cache_release()` (frees the linked list, RAM goes back
   to baseline) and `ps5ConnectEvent(true)` fires.
5. `ps5ConnectEvent` calls `ps5Enable()`, which sends the magic feature
   report `{0x53, 0xF4, 0x43, 0x02}` on the **control** channel — this
   tells the DualSense to start streaming BT 0x31 input reports.
6. First input report arrives at `ps5_l2cap_data_ind_cback` →
   `parsePacket()` writes every field directly onto `ps5` (sticks,
   buttons, gyro, accel, touchpad, status), then calls `ps5_mark_alive()`.
7. `ps5_mark_alive()` flips `g_active` true on its first call, fires the
   user's `attachOnConnect` callback (real "alive" moment), and on every
   subsequent call fires `attach`.

`isConnected()` auto-retries: when disconnected, it calls
`ps5_l2cap_reconnect()` at most every 5 seconds.

## DualSense Bluetooth wire format (verified vs Linux kernel)

### OUTPUT report (host -> controller, 79 bytes total)

Sent on the HID **interrupt** channel (PSM 0x13). Layout:

| Offset | Field                                                    |
|--------|----------------------------------------------------------|
|   0    | `0xA2` HID DATA\|OUTPUT header (covered by CRC)          |
|   1    | `0x31` report ID                                         |
|   2    | seq tag (high 4 bits = sequence, increments each frame)  |
|   3    | `0x10` (DualSense BT tag)                                |
|   4    | valid_flag0                                              |
|   5    | valid_flag1                                              |
|   6    | motor_right (small / high-frequency rumble)              |
|   7    | motor_left  (large / low-frequency rumble)               |
|  12    | mute LED (0=off, 1=on, 2=pulse)                          |
|  14    | R2 trigger mode + 10 param bytes [15..24]                |
|  25    | L2 trigger mode + 10 param bytes [26..35]                |
|  42    | valid_flag2                                              |
|  45    | lightbar setup byte                                      |
|  46    | LED brightness                                           |
|  47    | player LED bitmask (5 bits, bit0..bit4)                  |
|  48-50 | lightbar R / G / B                                       |
|  75-78 | CRC32 little-endian over bytes [0..74]                   |

Valid-flag bits that we set:

- `valid_flag0 |= 0x01` (compatible vibration)  ← rumble takes effect
- `valid_flag0 |= 0x02` (haptics select)
- `valid_flag0 |= 0x04` (R2 adaptive trigger enable)
- `valid_flag0 |= 0x08` (L2 adaptive trigger enable)
- `valid_flag1 |= 0x01` (mic-mute LED enable)
- `valid_flag1 |= 0x04` (lightbar enable)       ← RGB takes effect
- `valid_flag1 |= 0x10` (player LED enable)     ← player LEDs take effect
- `valid_flag2 |= 0x01` (light brightness enable) ← byte 46 (player LED brightness) takes effect

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HamzaYslmn/esp-ps5](https://github.com/HamzaYslmn/esp-ps5) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
