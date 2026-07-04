---
trigger: always_on
description: Project-specific guidance for Claude Code working in this repository. Read this before making changes.
---

# CLAUDE.md

Project-specific guidance for Claude Code working in this repository. Read this before making changes.

## What this project is

`openvlm` is a cross-platform CLI for reading, writing, and validating the EEPROM on **OpenVLM USB audio devices**. The hardware is a **C-Media CM108B** USB audio chip wired to a 93C46 SPI EEPROM, with a GPIO1 hardware strap that distinguishes OpenVLM-branded devices from generic CM108-family devices.

The CLI talks to the chip exclusively over USB-HID class control transfers (`Get_Input_Report` / `Set_Output_Report`) — there is no kernel driver, no vendor blob, and no platform-specific cable. The CM108B datasheet §7.4 documents the HID protocol; §7.1.3 documents the EEPROM word layout.

## Hardware context that shapes the code

These facts are not in the datasheet but are required to read the code correctly:

- **Stock chips ship with a blank EEPROM.** The strings `C-Media Electronics Inc.` / `USB Audio Device` come from the chip's internal ROM, not from EEPROM bytes. There is no "factory image" to read off a virgin device.
- **VID/PID are write-locked in the CLI.** Both are sourced from compiled-in constants (`cm108.OpenVLMVendorID = 0x0D8C`, `cm108.OpenVLMProductID = 0x0012`). Programming a different VID/PID would prevent this CLI from finding the device, so YAML, `update`, and per-field flags all refuse to set them.
- **Product/manufacturer strings are also write-locked** to the compiled-in `OpenVLMDefaults` (`OpenVLM` / `BuildsByShane`). Same reason — these are identity, not configuration.
- **GPIO1 strap is the OpenVLM identity probe.** A generic CM108-family device has GPIO1 floating low; an OpenVLM device pulls it high. `openvlm identify` and the `requireOpenVLM` gate on every write verb check this. `--force` bypasses for bench / bootstrap work.
- **macOS IOKit returns transient `kIOReturnError` (0xE00002BC)** under host-controller pressure during back-to-back HID reports. The protocol layer absorbs this with bounded retries; do not remove the `transferRetries` / `verifyRetries` machinery without a replacement.

## Datasheet pinning issues (open)

Three load-bearing assumptions in the codec are unverified against real hardware. They need a one-time bench gate (see [.claude/plans/review-the-last-implementation-wise-rossum.md](.claude/plans/review-the-last-implementation-wise-rossum.md) Phase G):

1. **`EEPROM_CTRL` bit layout.** Datasheet §7.4 names the register but never documents `op<<6 | addr` — that's the convention we copied from Linux `hid-cm108`.
2. **String body byte order in body words.** Datasheet specifies the header word's byte order, not the body words.
3. **Init-volume encoding (two's complement vs. offset-binary).** Currently two's complement; datasheet §8.3's step counts strongly imply offset-binary. Validator is narrowed defensively (intersection of doc range and 5/6/7-bit two's-complement faithful range) so values that would silently corrupt are rejected.

If you touch volume encoding or string parsing, read that plan file first.

## Architecture

```
cmd/                 — Cobra subcommands (one file per verb)
internal/cm108/      — OpenVLM-specific knowledge: VID/PID, GPIO1 probe, device selection
internal/eeprom/     — Datasheet §7.1.3 codec, validator, HID protocol helpers
internal/hidx/       — Cross-platform HID transport seam (linux/darwin/windows + fake)
```

Layering rule: `cmd` → `eeprom` + `cm108` → `hidx`. `hidx` knows nothing about CM108B or EEPROMs; `eeprom` knows nothing about USB enumeration or the GPIO1 strap; `cm108` is the bridge between the two.

### Subcommands ([cmd/](cmd/))

| Verb | Purpose |
|------|---------|
| `list` | enumerate every CM108-VID/PID device, report GPIO1 strap state |
| `identify` | exit 0 if selected device's GPIO1 reads high, exit 3 otherwise |
| `read` | dump 128-byte raw EEPROM image to file or stdout |
| `dump` | decode EEPROM, render as YAML / text / hex |
| `write` | program a 128-byte raw image or a YAML overrides file |
| `update` | read-modify-write a single field |
| `provision` | write `OpenVLMDefaults` with optional YAML or per-field flag overrides |
| `wipe` | erase EEPROM to all-`0xFF` (or `0x00`) — destructive, requires `--yes` |

Every write verb honors the GPIO1 safety gate; `--force` is the documented bypass.

### HID transport ([internal/hidx/](internal/hidx/))

`Transport` is single-threaded per device handle. `Backend` enumerates and opens.

- `transport_linux.go` — pure Go, hidraw ioctls (`HIDIOCGINPUT` / `HIDIOCSOUTPUT`) via `golang.org/x/sys/unix`. Walks `/sys/bus/usb/devices` over an `fs.FS` (testable with `fstest.MapFS`).
- `transport_darwin.go` — CGO via `github.com/sstallion/go-hid` (hidapi → IOKit). The **only** file in the repo that requires CGO. `make build` on macOS sets `CGO_ENABLED=1` automatically; Linux/Windows builds are pure Go and produce static binaries.
- `transport_windows.go` — pure Go via `setupapi.dll` / `hid.dll`.
- `fake.go` — in-memory backend used by every test in the repo. Implements the CM108B HID protocol closely enough that production code paths are exercised identically in tests as on hardware.

### EEPROM codec ([internal/eeprom/](internal/eeprom/))


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenMANET/OpenVLM](https://github.com/OpenMANET/OpenVLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
