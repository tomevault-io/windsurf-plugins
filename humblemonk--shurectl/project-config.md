---
trigger: always_on
description: The goal of this project is to build and maintain **shurectl**, an open-source terminal UI
---

# shurectl — Project Instructions

The goal of this project is to build and maintain **shurectl**, an open-source terminal UI
configurator for Shure USB audio interfaces (currently the **MVX2U** and **MV6**) on Linux
and macOS. It replaces the Windows/Mac-only ShurePlus MOTIV Desktop app by communicating
with devices directly over USB HID.

This is a Rust project. Operate as a senior Rust developer: write clean, readable, maintainable
code. Avoid clever abstractions. The simple, obvious solution is almost always correct.

---

## 🚨 AUTOMATED CHECKS ARE MANDATORY

ALL check failures are BLOCKING — everything must be ✅ GREEN before moving on.
No errors. No formatting issues. No linting problems. Zero tolerance.

**Reality checkpoint command:**
```
cargo clippy -- -D warnings && cargo fmt --check && cargo test
```

Run this after every complete feature, before starting a new component, and whenever
something feels wrong.

---

## CRITICAL WORKFLOW — ALWAYS FOLLOW THIS

**Research → Plan → Implement**

Never jump straight to coding. Always follow this sequence:

1. **Research**: Explore the codebase, understand existing patterns
2. **Plan**: Create a detailed implementation plan and confirm it with me
3. **Implement**: Execute with validation checkpoints

When asked to implement any feature, first say:
> "Let me research the codebase and create a plan before implementing."

For complex or architectural decisions, say:
> "Let me ultrathink about this before proposing a solution."

---

## Project Structure

This is a single-crate Rust binary. All source lives under `src/`:

```
src/
  main.rs       # Entry point, CLI args (--demo, --list), event loop, key handling
  app.rs        # Application state: Tab, Focus, DeviceState, DeviceAction events
  device.rs     # hidapi wrapper: open MVX2U, send/receive HID reports
  meter.rs      # cpal audio capture: real-time dBFS metering, RollingWindow, PeakWindow
  presets.rs    # Host-side preset storage: TOML serialisation, load/save/delete, PresetSlot
  protocol.rs   # Packet encoding, CRC-16/ANSI, all command constructors, apply_response()
  ui.rs         # ratatui rendering: all 5 tabs + help overlay
```

**Data flow:** key event → `handle_key()` → `DeviceAction` → `apply_action()` → `device.rs` → HID packet → `protocol.rs`

**Meter data flow:** cpal audio callback → `meter_level` (AtomicI32) + `peak_window` (Mutex<PeakWindow>) → `ui.rs` reads on each render tick

**Tab structure:** Main | EQ | Dynamics | Presets | Info

---

## shurectl Domain Rules

These are project-specific patterns learned from working in this codebase.

### USB HID Protocol

The MVX2U uses plain USB HID Output/Input Reports for all configuration. Every packet is
exactly 64 bytes, sent via `hid_write()` and received via `hid_read()`:

```
[0x01] [0x11] [0x22] [seq] [0x03] [0x08] [data_len] [0x70] [data_len] [cmd0][cmd1][cmd2] [feat_addr...] [value...] [crc_hi] [crc_lo] [0x00 padding...]
  ↑─── Report ID ───↑                                                                      ↑──────────── CRC-16/ANSI covers from 0x11 onward ───────────────↑
```

- **USB IDs**: VID `0x14ED`, PID `0x1013`
- **Header magic**: `0x11 0x22` — never changes
- **Report ID**: `0x01` — required as byte 0 by hidapi's `hid_write()`; our buffers are 65 bytes total (1 report ID + 64 payload)
- **CRC**: CRC-16/ANSI — poly `0x8005`, init `0x0000`, reflected input and output (NOT CCITT-FALSE)
- **Transport**: plain HID Output Reports (`hid_write`) for commands; Input Reports (`hid_read`) for responses — `HIDIOCSFEATURE`/`HIDIOCGFEATURE` are NOT used
- **Interface**: accessed via `/dev/hidrawN`, not the USB audio class interface
- **SET + CONFIRM**: every SET command must be immediately followed by a CONFIRM packet (`CMD_CONFIRM`); the device will not apply the change without it

All command byte values and feature address constants live in `protocol.rs`.
Do not hardcode raw byte values outside of `protocol.rs`.

### Adding a New Command

New device commands always follow this sequence — do not skip steps:

1. `protocol.rs` — add `FEAT_*` address constant, `cmd_get_*` and `cmd_set_*` constructor functions, and a matching branch in `apply_response()` to decode the GET response into `DeviceState`
2. `device.rs` — add typed `get_*` / `set_*` methods on `Mvx2u` that call the constructors; add `cmd_get_*` to the `getters` slice in `get_state()` if it's part of full state readback
3. `app.rs` — add a `DeviceAction` variant if user-triggerable; update `adjust_focused()` or `toggle_focused()` as appropriate
4. `main.rs` — handle the new variant in `apply_action()`
5. `ui.rs` — add UI element if needed
6. `protocol.rs` tests — add a roundtrip test for the new packet

### Protocol Verification

If a command doesn't behave as expected on the real device, **capture packets first**:

```bash
sudo modprobe usbmon
lsusb | grep -i shure          # find bus number
sudo wireshark -i usbmonN      # replace N with bus number
# Filter: usb.transfer_type == 0x01
```

Compare the captured bytes to what the `cmd_*` constructors in `protocol.rs` produce.
Feature address constants (`FEAT_*`) and value encoding are the most likely source of

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Humblemonk/shurectl](https://github.com/Humblemonk/shurectl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
