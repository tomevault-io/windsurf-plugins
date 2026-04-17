---
trigger: always_on
description: Upload safety depends on the platform. ESP32-S3 and nRF52840 use completely different upload protocols.
---

# Claude Code Instructions for Blinky Project

## CRITICAL: Upload Safety

Upload safety depends on the platform. ESP32-S3 and nRF52840 use completely different upload protocols.

### ESP32-S3: `arduino-cli upload` is SAFE

```bash
# MUST use full FQBN with USBMode=hwcdc — see note below
arduino-cli compile --upload --fqbn 'esp32:esp32:XIAO_ESP32S3:USBMode=hwcdc,CDCOnBoot=default,MSCOnBoot=default,DFUOnBoot=default,UploadMode=default,CPUFreq=240,PSRAM=opi' -p /dev/ttyACM0 blinky-things
```

`arduino-cli upload` on ESP32-S3 calls **esptool**, which talks to the chip's hardware ROM bootloader. The ROM bootloader is burned into silicon and cannot be bricked. esptool verifies every write. If interrupted, the ROM bootloader still works and you just re-flash.

### ESP32-S3: JTAG/PDM Pin Conflict

**GPIO42 (PDM CLK) and GPIO41 (PDM DATA) are also JTAG strap pins (MTMS/MTDI).**

ESP32 core 3.3.7 requires `USBMode=hwcdc` for serial (the default TinyUSB mode has unresolved `HWCDCSerial` linker errors — core bug). This enables the `USB_SERIAL_JTAG` peripheral which may claim GPIO42/41 at boot, silently blocking the PDM microphone.

**Mitigation:** `Esp32PdmMic::begin()` calls `gpio_reset_pin()` on both PDM pins before I2S init, then verifies data flows with a 500ms blocking read. If verification fails, `begin()` returns false and the boot log reports `"Audio controller failed to start"`.

**If the ESP32 core is upgraded**, re-test PDM mic on ESP32-S3. If the TinyUSB linker bug is fixed in a future core version, switch back to the default FQBN (`esp32:esp32:XIAO_ESP32S3`) which avoids the JTAG peripheral entirely.

### nRF52840: NEVER use `arduino-cli upload`

**NEVER use `arduino-cli upload` or `adafruit-nrfutil dfu serial` on nRF52840!**

- `arduino-cli upload` calls `adafruit-nrfutil dfu serial` under the hood
- This protocol has race conditions in USB port re-enumeration
- Single-bank DFU can leave firmware partially written
- Can brick the device, requiring SWD hardware to recover

**Use UF2 instead:**
```bash
make uf2-upload UPLOAD_PORT=/dev/ttyACM0
```

**Why UF2 is safe:**
- Invalid/corrupt UF2 files are silently rejected (cannot brick)
- Simple file copy to mass storage (no serial protocol race conditions)
- Bootloader protects itself (hardware-enforced)
- If interrupted, old firmware stays intact

**Firmware upload is managed by blinky-server** (no standalone scripts):
```bash
# Compile and flash a single device
curl -X POST http://blinkyhost.local:8420/api/firmware/compile
curl -X POST http://blinkyhost.local:8420/api/devices/{id}/flash \
  -H 'Content-Type: application/json' \
  -d '{"firmware_path": "/tmp/blinky-build/blinky-things.ino.hex"}'

# Flash ALL connected nRF52840 devices
curl -X POST http://blinkyhost.local:8420/api/fleet/flash \
  -H 'Content-Type: application/json' \
  -d '{"firmware_path": "/tmp/blinky-build/blinky-things.ino.hex"}'
```

The server owns all serial/BLE connections — no port contention, no lock files, no race conditions. It handles bootloader entry, UF2 copy/BLE DFU transfer, and automatic reconnection.

### Safe Operations Summary

**ESP32-S3:**
- `arduino-cli compile --upload` — Safe (uses esptool)
- `arduino-cli upload` — Safe (uses esptool)

**nRF52840:**
- `arduino-cli compile` — Safe (compile only)
- `make uf2-upload` — Safe (uses mass storage, not DFU serial)
- `make uf2-check` — Safe (dry run, no upload)
- `arduino-cli upload` — **NEVER** (uses fragile DFU serial protocol)
- `adafruit-nrfutil dfu serial` — **NEVER**

**Both platforms:**
- `arduino-cli core list/install` — Safe
- Reading serial ports — Safe

### Test Chips (Unenclosed) — Use First for Risky Firmware

There are bare (unenclosed) chips attached to blinkyhost. **Always test untested bootloader changes or risky firmware on these first.**

- Reset button is directly accessible — no disassembly required
- SWD recovery pads are accessible — no emergency disassembly in worst case
- Safe to experiment on before flashing any installed device

### If a Device Becomes Unresponsive

**nRF52840** devices are physically installed and reset buttons are NOT accessible.
If a device stops responding to serial commands:
1. Try power-cycling via USB hub: `uhubctl -a cycle -p <port>`
2. Use blinky-server: `curl -X POST http://blinkyhost.local:8420/api/devices/{id}/flash -d '{"firmware_path":"/tmp/blinky-build/blinky-things.ino.hex"}'`
3. If the port disappeared entirely, wait 10 seconds and check `ls /dev/ttyACM*`
4. Last resort: physically access the device and double-tap reset

## CRITICAL: Long-Running Scripts

**NEVER run ML training or other long-running scripts as Claude session tasks.**
Claude background tasks die when the session ends, killing training mid-run.

Always use tmux:
```bash
tmux new-session -d -s training "cd ml-training && source venv/bin/activate && PYTHONUNBUFFERED=1 python train.py --config configs/frame_fc.yaml --output-dir outputs/<experiment_name> 2>&1 | tee outputs/<experiment_name>/training.log"
```

To check progress: `tmux attach -t training` or `tail -f ml-training/outputs/<experiment_name>/training.log`

`train.py` enforces this — it will refuse to start outside tmux/screen unless `--allow-foreground` is passed.

## Firmware Versioning


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jdubz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
