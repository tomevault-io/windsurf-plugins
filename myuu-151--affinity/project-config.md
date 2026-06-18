---
trigger: always_on
description: All editor authoring — HUD element x/y, piece localX/localY, animation keyframe offsets, sprite world positions — is in **GBA units (240 wide × 160 tall)**. The NDS port renders content at the same coords with extra blank space on the right/bottom of its 256×192 screen.
---

# Affinity Editor — Development Conventions

## Editor canvas is GBA-native (240×160)

All editor authoring — HUD element x/y, piece localX/localY, animation keyframe offsets, sprite world positions — is in **GBA units (240 wide × 160 tall)**. The NDS port renders content at the same coords with extra blank space on the right/bottom of its 256×192 screen.

**When porting NDS code that does positional math against "screen width/height" for editor-authored content, use 240/160, not 256/192.** Common bite: HUD anim keyframes like `+240, +0` mean "slide one full GBA screen and wrap back to start." Wrapping that at the NDS width (256) leaves a 16 px gap. The same rule applies vertically (use 160 for any future Y wrap).

Per-frame HUD anim ticking matches GBA's 60 Hz; the editor's FPS slider value N translates to `speed = 60/N` frames per tick.

## Visual Script Node Code Previews

Every action node in `src/editor/frame_loop.cpp` has a `setActionFunc()` call that defines the **actual C implementation** shown in the editor's "Mode 0 Runtime" code window. This is NOT a preview — it is the real generated code.

### Runtime Chain Convention

Every node's `setActionFunc` body MUST include:

1. **The generated C code** — the actual lines emitted into `mapdata.h` (e.g. `afn_player_frozen = 1;`)
2. **`// --- Runtime (main.c) ---`** — a separator comment
3. **How main.c consumes those variables** — showing the full chain of what happens at runtime

Example (Sprint node):
```c
setActionFunc(infoNode, "_sprint",
    "    afn_move_speed = 37;\n"
    "    // --- Runtime (main.c) ---\n"
    "    // Mode 0: tm_move_frames = 48 / afn_move_speed; // = 1 frames/tile\n"
    "    //         tm_move_timer = tm_move_frames;\n"
    "    //         px = lerp(fromX, toX, t / tm_move_frames);\n"
    "    // Mode 4: moveSpeed = afn_move_speed; // = 37\n"
    "    //         player_x += (viewSin * inputFwd * moveSpeed) >> 16;");
```

### Rules

- **Always show both Mode 0 and Mode 4 runtime paths** when the variable is used differently in each mode
- **Keep runtime comments to 2-4 lines** — concise, showing the key consumption points
- **Pure math/data nodes** (Add, Sub, Mul, etc.) that just `return` a value: add `// --- Runtime --- inline data node, evaluated at call site`
- **Gate/flow nodes** (IsMoving, DoOnce, etc.): add a brief comment about how the condition is evaluated
- **When adding a new node**, always include the runtime chain — search `main.c` for how the variable is consumed
- **When modifying node behavior** (e.g. FreezePlayer now also sets `afn_play_anim = -1`), update BOTH the exporter (`gba_package.cpp`) AND the `setActionFunc` body in `frame_loop.cpp`. This applies to ANY change — codegen, runtime consumption, or guard conditions. Do this in the SAME edit pass, never defer it. If you touch `gba_package.cpp` or `main.c` for a node, you MUST also touch the `setActionFunc` before moving on.

### Key Files

- `src/editor/frame_loop.cpp` — Node code previews (`setActionFunc` calls, ~line 11500+)
- `src/platform/gba/gba_package.cpp` — GBA exporter (generates `mapdata.h` from nodes)
- `gba_runtime/source/main.c` — GBA runtime (consumes the generated variables)
- `gba_runtime/include/mapdata.h` — Generated header (output of exporter, read-only)

### GBAScriptNodeType Enum Ordering

**NEVER insert new entries in the middle of the `GBAScriptNodeType` enum** in `gba_package.h`. Always append new node types immediately before `COUNT`. Inserting in the middle shifts all subsequent integer values, which breaks every saved project file that references those node types.

If a middle insertion has already shipped (e.g. `IsFalling` at position 36), it must stay — add **version-based migration** instead:
1. Bump the save version in `fprintf(f, "version=N\n")`
2. At **all 4 node-load sites** in `frame_loop.cpp` (blueprint nodes, Mode 0 scene nodes, Mode 4 scene nodes, Mode 7 scene nodes), add: `if (projectVersion < N && typeInt >= POS) typeInt++;`
3. The `VsNodeType` enum in `frame_loop.cpp` must match `GBAScriptNodeType` in `gba_package.h` exactly

### Export Ordering in mapdata.h

HUD data arrays (`afn_hud_elems`, `afn_hud_stops`, etc.) are emitted BEFORE script and blueprint functions so that ShowHUD/CursorUp/CursorDown can reference them. Do not move the HUD section after the script/blueprint codegen sections.

## GBA Runtime Debugging (mGBA + GDB)

The GBA runtime is opaque on freezes / crashes — no console, no logs unless you instrument them. Always use mGBA's GDB stub before guessing at fixes; static analysis is unreliable for ISR/timing bugs.

### Connecting

1. In mGBA: **Tools → Start GDB server** → leave defaults (port 2345, bind 127.0.0.1) → click **Start**. The dialog stays open while the server runs; closing it stops the server.
2. Load the `.gba` ROM in mGBA (or have it already loaded).
3. In a terminal in `gba_runtime/`:
   ```
   "C:\devkitPro\devkitARM\bin\arm-none-eabi-gdb.exe" affinity.elf
   ```
   (Plain `arm-none-eabi-gdb` isn't on PATH; full path always works.)
4. At the `(gdb)` prompt:
   ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [myuu-151/Affinity](https://github.com/myuu-151/Affinity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
