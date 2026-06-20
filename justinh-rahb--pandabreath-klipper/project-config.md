---
trigger: always_on
description: Klipper extras module to integrate the **BIQU Panda Breath** smart chamber heater/filter with **Klipper-based printers** (primary target: Snapmaker U1).
---

# pandabreath-klipper

Klipper extras module to integrate the **BIQU Panda Breath** smart chamber heater/filter with **Klipper-based printers** (primary target: Snapmaker U1).

## Project Goal

The Panda Breath has no native Klipper support yet. BTT has not released the firmware source. Three parallel strategies are in development:

1. **Stock firmware path** — Klipper `extras/` module that speaks the device's WebSocket JSON API. Target firmware: v0.0.0 (only confirmed stable release; v1.0.1+ has thermal/timing bugs including removal of PTC thermal runaway detection in v1.0.2).
2. **ESPHome path** — Reflash the ESP32-C3 with ESPHome, which provides native TRIAC phase-angle fan speed control (`ac_dimmer` component), configurable PTC heater relay, NTC sensors, and restored thermal runaway protection. ESPHome integration with Klipper via MQTT.
3. **KlipperMCU path** — Reflash the ESP32-C3 with a custom ESP-IDF firmware that speaks the Klipper MCU binary protocol over UART0 (via the onboard CH340K USB-C bridge). The Panda Breath becomes a native `[mcu panda_breath]` — no Python extras module, no MQTT broker, Klipper's own PID and thermal safety apply directly. Fan control is internal to firmware (TRIAC phase-angle via zero-crossing ISR). See `klipper-firmware/`.

The Klipper module (`panda_breath.py`) supports both via a transport abstraction: `firmware: stock` uses the WebSocket transport; `firmware: esphome` uses the MQTT transport. From Klipper's perspective the interface is identical either way.

## Device: BIQU Panda Breath

**Hardware** (schematic reverse-engineered from real device — see [research/hardware-schematic.md](research/hardware-schematic.md))
- Controller: ESP32-C3 (WiFi 2.4GHz; USB-C flashing via CH340K)
- Heater: 300W PTC, relay-switched (MGR-GJ-5-L SSR, on/off only; firmware duty-cycles for regulation)
- Fans: 2× 75×75×30mm, TRIAC phase-angle speed control (BT136-800E + MOC3021S + zero-crossing)
- Chamber temp: NTC 100K thermistor on dedicated ADC channel (33K 0.1% divider) — this is `warehouse_temper`
- PTC temp: second NTC 100K on separate ADC channel — for thermal runaway protection only
- Buttons: 4× LED-backlit tactile switches (K1–K4); K1–K3 are K6-6140S01 with per-button LEDs
- AC input: 110–220V → HLK-PM01 (5V) → AS1117-3.3 (3.3V MCU)
- Logic: 3.3V

**Firmware**
- Current release: V1.0.2 (buggy — V1.0.1+ has thermal/timing issues)
- V0.0.0 (Aug 25 2025) is the only confirmed stable version
- Source not yet published; BTT tracking: https://github.com/bigtreetech/Panda_Breath
- License: CC-BY-NC-ND-4.0 (non-commercial)
- OTA update via HTTP POST to `/ota` endpoint (not WebSocket); max app size 0x480000 bytes
- Full 4MB flash dump of V0.0.0 obtained via `esptool flash_read` from a real device — contains embedded web UI JS which is the protocol source of truth

**Network**
- Default hostname: `PandaBreath.local`
- AP fallback SSID: `Panda_Breath_XXXXXXXXXX`, password: `987654321`, IP: `192.168.254.1`
- Control interface: WebSocket at `ws://<ip>/ws` (port 80, no auth)

## WebSocket Protocol

All messages are JSON with a top-level `settings` key. The device sends state updates; you send commands in the same format.

### Inbound (device → client) — state updates
```json
{ "settings": { "work_on": true } }
{ "settings": { "work_mode": 2 } }
{ "settings": { "hotbedtemp": 60 } }
{ "settings": { "temp": 45 } }
{ "settings": { "warehouse_temper": 38.5 } }
```

### Outbound (client → device) — commands
```json
{ "settings": { "work_on": true } }          // enable/disable device
{ "settings": { "work_mode": 1 } }           // 1=auto, 2=always_on, 3=filament_drying
{ "settings": { "set_temp": 45 } }           // set target temperature (°C)
{ "settings": { "hotbedtemp": 60 } }         // hotbed temp that triggers auto mode
```

### Field reference (confirmed from binary strings + community)

| Field | Dir | Type | Description |
|---|---|---|---|
| `work_on` | ↔ | bool | Power on/off |
| `work_mode` | ↔ | int | 1=Auto (follows bed temp), 2=Always On, 3=Filament Drying |
| `hotbedtemp` | ↔ | int | Bed temp threshold that triggers auto mode |
| `warehouse_temper` | ←device | float | Current chamber temperature reading |
| `set_temp` | →device | int | Writable field to set target chamber temperature (Confirmed) |
| `temp` | ←device | int | Target temp readback (may be read-only) |
| `filtertemp` | ? | int | Filter temperature (threshold or sensor reading — TBD) |
| `filament_temp` | ↔ | int | Filament drying target temperature |
| `filament_timer` | ↔ | int | Filament drying duration |
| `filament_drying_mode` | ←device | bool/int | Filament drying active flag |
| `custom_temp` | ↔ | int | Custom mode temperature |
| `custom_timer` | ↔ | int | Custom mode timer |
| `remaining_seconds` | ←device | int | Countdown timer (drying mode) |
| `fw_version` | ←device | string | Firmware version string |
| `ptc_sensor_status` | ←device | int | PTC thermistor health |
| `warehouse_sensor_status` | ←device | int | Chamber thermistor health |
| `ptc_heater_status` | ←device | int | PTC heater element status |
| `cal_ptc_temp` | ←device | float | Calibrated PTC temperature |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [justinh-rahb/pandabreath-klipper](https://github.com/justinh-rahb/pandabreath-klipper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
