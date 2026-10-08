---
trigger: always_on
description: Linux daemon that implements the ROLI Blocks protocol to keep devices alive, control LEDs, and manage topology. ROLI devices require an active host-side handshake over MIDI SysEx to enter "API mode": without it, they show a "searching" animation and eventually power off.
---

# blocksd: ROLI Blocks Linux Daemon

## Project Overview

Linux daemon that implements the ROLI Blocks protocol to keep devices alive, control LEDs, and manage topology. ROLI devices require an active host-side handshake over MIDI SysEx to enter "API mode": without it, they show a "searching" animation and eventually power off.

**Stack:** Python 3.13+, asyncio, python-rtmidi, Typer, Rich, Pydantic
**Package Manager:** uv
**Linter:** ruff
**Type Checker:** ty

## Architecture

```
daemon.py                 asyncio main loop, signal handling
  └─ TopologyManager      polls MIDI ports every 1.5s
       └─ DeviceGroup     per-USB lifecycle (serial → topology → API mode → ping)
            └─ MidiConnection   python-rtmidi wrapper (send/receive SysEx)
                 └─ protocol/*   pure protocol logic (packing, builder, decoder)
```

## ROLI Blocks Protocol Reference

### SysEx Framing

All BLOCKS protocol messages are MIDI SysEx:

```
F0 00 21 10 77 [deviceIndex] [7-bit packed payload] [checksum] F7
│  │           │  │                                  │
│  │           │  └ lower 6 bits = topology index    └ payload checksum & 0x7F
│  │           │    bit 6: 0=host→device, 1=device→host
│  │           └ BLOCKS product byte
│  └ ROLI manufacturer ID
└ SysEx start
```

### Checksum Calculation

```python
def calculate_checksum(data: bytes) -> int:
    checksum = len(data) & 0xFF
    for byte in data:
        checksum = (checksum + (checksum * 2 + byte)) & 0xFF
    return checksum & 0x7F
```

### 7-Bit Packing

Payload bits are packed LSB-first across 7-bit bytes (bit 7 always 0 for MIDI safety). When a value spans byte boundaries, low bits fill the current byte remainder, high bits continue in the next byte's low bits.

### Serial Number Request (separate SysEx format)

- **Request:** `F0 00 21 10 78 3F F7`
- **Response header:** `F0 00 21 10 78`
- Response contains MAC prefix `48:B6:20:` followed by 16-char serial
- Serial prefixes identify device type:
  - `LPB`/`LPM` = Lightpad Block
  - `SBB` = Seaboard Block
  - `LKB` = LUMI Keys Block
  - `LIC` = Live Block
  - `LOC` = Loop Block
  - `DCB` = Developer Control Block
  - `TCB` = Touch Block

### USB Device IDs (Vendor `0x2AF4`)

| PID      | Device              |
| -------- | ------------------- |
| `0x0100` | Seaboard (original) |
| `0x0200` | Seaboard RISE 25    |
| `0x0210` | Seaboard RISE 49    |
| `0x0700` | Seaboard Block      |
| `0x0900` | Lightpad Block      |
| `0x0E00` | LUMI Keys / Piano M |
| `0x0F00` | ROLI Piano (49-key) |
| `0x1000` | ROLI Airwave Pedal  |

### Protocol Constants

```
ROLI_SYSEX_HEADER     = F0 00 21 10 77
SERIAL_DUMP_REQUEST   = F0 00 21 10 78 3F F7
SERIAL_RESPONSE_HDR   = F0 00 21 10 78
RESET_MASTER          = F0 00 21 10 49 F7
PROTOCOL_VERSION      = 1
TOPOLOGY_INDEX_BROADCAST = 63
API_MODE_PING_TIMEOUT_MS = 5000
```

### Message Types: Device → Host

| ID     | Name                   | Payload                                         |
| ------ | ---------------------- | ----------------------------------------------- |
| `0x01` | deviceTopology         | 7b deviceCount, 8b connectionCount, then blocks |
| `0x02` | packetACK              | 10b packetCounter                               |
| `0x03` | firmwareUpdateACK      | 7b code, 32b detail                             |
| `0x04` | deviceTopologyExtend   | continuation of topology                        |
| `0x05` | deviceTopologyEnd      | signals end of multi-packet topology            |
| `0x06` | deviceVersion          | index + version string                          |
| `0x07` | deviceName             | index + name string                             |
| `0x10` | touchStart             | 7b devIdx, 5b touchIdx, 12b x, 12b y, 8b z      |
| `0x11` | touchMove              | same as touchStart                              |
| `0x12` | touchEnd               | same as touchStart                              |
| `0x13` | touchStartWithVelocity | + 8b vx, 8b vy, 8b vz                           |
| `0x14` | touchMoveWithVelocity  | + velocity                                      |
| `0x15` | touchEndWithVelocity   | + velocity                                      |
| `0x18` | configMessage          | config command + data                           |
| `0x20` | controlButtonDown      | 7b devIdx, 12b buttonID                         |
| `0x21` | controlButtonUp        | 7b devIdx, 12b buttonID                         |
| `0x28` | programEventMessage    | 3 × 32b integers                                |
| `0x30` | logMessage             | string data                                     |

### Message Types: Host → Device

| ID     | Name                 | Payload                            |
| ------ | -------------------- | ---------------------------------- |
| `0x01` | deviceCommandMessage | 9b command (see Device Commands)   |
| `0x02` | sharedDataChange     | 16b packetIndex + data change cmds |
| `0x03` | programEventMessage  | 3 × 32b integers                   |
| `0x04` | firmwareUpdatePacket | 7b size + 7-bit encoded data       |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyperb1iss/blocksd](https://github.com/hyperb1iss/blocksd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
