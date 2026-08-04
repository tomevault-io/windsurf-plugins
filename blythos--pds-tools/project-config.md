---
trigger: always_on
description: This file is the persistent context document for Claude Code sessions. Read it in full at the start of every session. It supersedes any inconsistency in other docs.
---

# Panzer Dragoon Saga — Decompilation & Reengineering Project (CLAUDE.md)

This file is the persistent context document for Claude Code sessions. Read it in full at the start of every session. It supersedes any inconsistency in other docs.

---

## Project Goal

Full decompilation and native modern port of Panzer Dragoon Saga (Sega Saturn, 1998, Team Andromeda). The end deliverable is an application that loads the original disc image and runs the complete game on modern hardware — not emulation, a true reimplementation.

The working approach has three reinforcing tracks:
1. **Asset extraction** — parse every data format on disc into usable modern representations
2. **Code decompilation** — convert SH-2 machine code to readable C/C++
3. **Reimplementation** — build a modern engine that runs the decompiled logic with the original assets

Asset extraction is the validation layer: every format we crack proves our understanding of the code that processes it.

Primary disc: Panzer Dragoon Saga (USA) Disc 1, raw Mode 2 track image (`.bin`).

---

## Multi-Agent Workflow

**Claude (this session)** handles:
- Core pipeline: MCB/CGB parsing, skeletal transforms, texture decoding
- Format reverse engineering and binary analysis
- Architecture decisions, project documentation
- Any task touching established binary format assumptions

**Gemini via Antigravity** handles self-contained tasks defined in `TASK_*.md` files committed to the repo. Antigravity reads context from `.antigravity/rules.md` and `docs/`. It does not have session memory.

**Critical rule:** Never let Antigravity modify the core MCB/CGB parsing pipeline or skeletal transform math without Claude reviewing first. Antigravity has broken working implementations before by "fixing" things it didn't understand (see CPK audio incident in session history).

---

## Key Reference: yaz0r's Azel Project

Primary format reference: `github.com/yaz0r/Azel` — MIT-licensed partial C++ reimplementation of the PDS engine, 468 commits, 2015–2020, open-sourced February 2026.

Use it as a **spec reference**, not a dependency. Read the C++ to understand binary formats; write our own Python tools.

Key source paths:
- `AzelLib/processModel.h/cpp` — 3D model parsing, quad format, pointer patching
- `AzelLib/dragonData.cpp` — Dragon filename tables
- `AzelLib/field/` — Field area loading
- `AzelLib/3dEngine_textureCache.cpp` — VDP1 VRAM and texture management
- `AzelLib/kernel/fileBundle.h` — MCB bundle structure
- `AzelLib/kernel/animation.cpp` — Animation track stepping, bone transform accumulation
- `AzelLib/town/townScript.cpp` — PRG bytecode interpreter (opcodes for subtitles, FMV)
- `AzelLib/common.h` — s_MCB_CGB struct, dragon config tables

---

## Saturn Hardware Architecture

- **CPU**: Dual Hitachi SH-2 (32-bit RISC, **big-endian**)
- **VDP1**: Sprite/polygon processor — all 3D geometry as textured quads
- **VDP2**: Background/tilemap processor — 2D backgrounds, scroll planes
- **VDP1 VRAM**: 512KB at `0x25C00000–0x25C7FFFF`
- **VDP2 Color RAM**: Palette data used by both VDP1 and VDP2
- **All data is big-endian** — use `struct.unpack('>...')` everywhere

---

## Disc Image Format

Raw Mode 2 CD-ROM:
- 2352 bytes per sector: 16-byte header + 2048 bytes data + 288 bytes ECC
- Standard ISO9660 filesystem, PVD at sector 16
- Files are in a flat root directory (no subdirectories on Disc 1)

```python
SECTOR_SIZE = 2352
SECTOR_HEADER = 16
SECTOR_DATA = 2048

def read_sector(f, sector_num):
    f.seek(sector_num * SECTOR_SIZE)
    raw = f.read(SECTOR_SIZE)
    if len(raw) < SECTOR_SIZE:
        return b'\x00' * SECTOR_DATA
    return raw[SECTOR_HEADER:SECTOR_HEADER + SECTOR_DATA]

def read_file_from_disc(f, start_sector, size):
    data = bytearray()
    remaining = size
    sector = start_sector
    while remaining > 0:
        chunk = read_sector(f, sector)
        take = min(remaining, SECTOR_DATA)
        data.extend(chunk[:take])
        remaining -= take
        sector += 1
    return bytes(data)
```

ISO9660 parsing: PVD at sector 16, root directory record at PVD offset 156, parse directory entries (record length byte, then name, sector, size fields). Little-endian for ISO9660 metadata fields only; game data is big-endian.

---

## Binary Read Helpers (use these everywhere)

```python
import struct

def ru8(data, off):  return data[off]
def ru16(data, off): return struct.unpack_from('>H', data, off)[0]
def rs16(data, off): return struct.unpack_from('>h', data, off)[0]
def ru32(data, off): return struct.unpack_from('>I', data, off)[0]
def rs32(data, off): return struct.unpack_from('>i', data, off)[0]
```

---

## File Formats (Established)

### MCB — Model/Character Block

Self-contained binary bundle loaded to Work RAM. Contains all data needed to render a character or object.

**Structure:**
```
[pointer table: N × u32 big-endian offsets]
[sub-resources at those offsets]
```

The pointer table ends where the first pointed-to data begins. Find the table length by: start at offset 0, read u32 values, stop when one falls outside the file or lands before the current scan position.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blythos/pds-tools](https://github.com/blythos/pds-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
