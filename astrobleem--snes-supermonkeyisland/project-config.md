---
trigger: always_on
description: Project-specific quickstart for AI coding agents (and humans pretending to be
---

# Agent guide for SNES-SuperMonkeyIsland

Project-specific quickstart for AI coding agents (and humans pretending to be
one). For codebase conventions and build commands, read `CLAUDE.md` first.
This file documents the **Mesen-MCP harness** — the highest-leverage tool in
the project, and the one most agents reach for wrong on first try.

## TL;DR

The project ships a patched Mesen2 emulator (`mesen/Mesen.exe`) with a
built-in MCP server. The fork's source + protocol docs live in a separate
repo (path varies by checkout; conventionally cloned as a sibling
directory). All you typically need is `tools/mcp_client.py` and
`mesen/Mesen.exe` from this repo. From Python:

```python
import sys; sys.path.insert(0, 'tools')
from mcp_client import McpSession

with McpSession(rom='distribution/SuperMonkeyIsland.sfc') as m:
    m.run_frames(500)
    state = m.get_state()         # frame counter, paused state
    m.pause()                      # always before multi-call inspection
    print(m.read_memory('snesMemory', 0x7EF967, 1).hex())  # currentRoom
```

Full tool surface is versioned with the deployed Mesen fork (46 tools in this
checkout; older notes may still say 30): `read_memory`, `write_memory`,
`add_exec_hook` / `add_read_hook` / `add_write_hook` / `add_frame_hook`,
`take_screenshot`, `crop_screenshot`, `record_audio`, `get_audio_state`,
`save_state_slot` / `load_state_slot`, `lookup_symbol`, `disassemble`,
`run_until`, `read_dma_state`, `get_ppu_state`, `reset_emulator` …
The Mesen-MCP fork's `AGENTS.md` is the canonical protocol doc; if you
have its source checked out as a sibling directory, look there. Otherwise
read it on GitHub (see `mesen/README.txt` for the repo URL pinned to
this build).

## When to reach for what

| Situation | First tool to try |
|---|---|
| "Boot to a known room, screenshot it" | `tools/room_test.py 33 -x 346 -y 133` (existing harness) |
| "Inspect SCUMM state mid-flow" | McpSession + `pause` + `read_memory` at SCUMM symbols |
| "Why does this differ between two boot paths?" | `tools/room33_real_diff.py` style state-diff (CGRAM, OAM, VRAM, DMA) |
| "When does this function execute?" | `add_exec_hook` on the lookup_symbol address |
| "Is the song right?" | `record_audio` then `tools/audio_analyze.py /path/to.wav --ref reference.wav` |
| "Drive the natural intro flow" | `set_input(BTN_START, 40)` in a loop with `set_input(0, 760)` between bursts |

## Project SCUMM constants worth knowing

```
SCUMM.currentRoom        $7EF967  byte
SCUMM.newRoom            $7EF969  byte
SCUMM.cutsceneNest       $7EF965  byte
SCUMM.actors.1           $7E890C  16-byte struct: {room, costume, x, y, facing, elev, moving, vis, ...}
GLOBAL.room.cameraX      $7EFA1F  word
GLOBAL.room.hasBg2Mask   $7EFA94  byte
xScrollBG1               $7EFD54  word
yScrollBG1               $7EFD56  word
MainScreen               $7EFD47  byte (mirrors $212C TMAIN)
```

Addresses shift on every rebuild. Use `lookup_symbol(SYM_FILE, '^name$')` for
anything not in this list. Never hard-code addresses you got from a prior
session.

## The five-minute checklist before doing real work

1. **Always pause before a multi-call inspection.** Two `read_memory` calls
   50ms apart see different states because the emulator runs at max speed
   between them. Either `m.pause()` first, or wrap reads in a single
   `with-pause` block.

2. **`run_frames(N)` is wall-clock approximate.** At MaximumSpeed flag,
   the emulator runs many more than N frames in N×16ms. If you need an
   exact frame count, use `m.get_state()['frameCount']` for ground truth
   or compose with `run_until(maxFrames=N)`.

3. **Drain stderr.** Mesen emits hundreds of `[CPU] Uninitialized memory
   read: ...` warnings during the first ~12k frames. If you spawn Mesen
   with `subprocess.PIPE` for stderr and don't drain it, the pipe fills,
   blocks Mesen's logger, and deadlocks `initialize`. The provided
   `McpSession` does this for you; copy that pattern if you roll your
   own.

4. **Filter hot hooks.** `add_exec_hook(0xC40026)` on `_scummvm.fetchLoop`
   fires hundreds of times per frame. Use `match_value` + `match_value_mask`
   for value filtering, or `add_frame_hook(every_n=N)` for periodic
   pulses, instead of trying to drain a flood of notifications.

5. **A silent WAV is a real observation, not a tool failure.**
   Cross-check with `get_audio_state()` — `keyOn != 0` or any voice with
   `envelope > 0` means actual playback. If both are zero, the song
   isn't playing yet (game-state issue, not MCP).

## Existing diagnostics that already use this

| Script | What it does |
|---|---|
| `tools/room_test.py` | Parametric room loader: pokes SCUMM newRoom, settles, screenshot. Original Lua-based, still useful. |
| `tools/room33_natural_flow.py` | Drives the real intro path with `set_input` + START spam. Reproduces the original room-33 corruption bug. |
| `tools/room33_real_diff.py` | Captures CGRAM, OAM, BG1 tilemap+tiles, BG2, scroll WRAM into both clean and corrupt paths and diffs. |
| `tools/room33_ppu_diff.py` | Compares full PPU state (layer enables, scroll, window mask). |
| `tools/room33_dma_regs.py` | Decodes all 8 DMA channels' control/source/target. |
| `tools/room33_bg2_diff.py` | Targeted WRAM dump of BG2-related state. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [astrobleem/SNES-SuperMonkeyIsland](https://github.com/astrobleem/SNES-SuperMonkeyIsland) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
