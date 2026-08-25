---
trigger: always_on
description: > A faithful, modern recreation of Garry Kitchen's GameMaker (Activision, 1985).
---

# GMC64 Technical Documentation

> A faithful, modern recreation of Garry Kitchen's GameMaker (Activision, 1985).

gmc64 lets you build, run, and edit C64-style games in the browser, with original-format .PRG, .SPR, .SND, .SNG, and .PIC files stored on D64 disk images. Files round-trip cleanly with the original 1985 GameMaker disk running on real C64 hardware.

This document is the internal engineering reference — architecture, file formats, gotchas, and recipes for working in the codebase. Reader-facing overview lives in `README.md`.

## Quick Reference

| Item | Value |
|------|-------|
| Music tempo | `gmMusic.BASE_BPM = 140` (tempo 80 = 140 BPM) |
| Screen resolution | 320×200 native, CSS scales to display |
| Coordinate system | GM fat pixels: X 12-171, Y 50-249 → 320×200: `(x-12)*2, y-50` |
| Movement formula | X: `speed / 46`, Y: `speed / 31.5` at 60fps (C64 pixels aren't square) |
| Animation formula | `frame_skip = 32 - animSpeed` |
| File separator | `/` not `.` (PETSCII): `PLAYER/SPR` not `PLAYER.SPR` |

## Project Structure

```
├── editor.html          Program editor/runner (AST-based VM)
├── sprite-maker.html    Sprite editor
├── scene-maker.html     Scene editor
├── sound-maker.html     Sound effect editor
├── music-maker.html     Music editor
├── js/
│   ├── gmRuntime.js     VM execution, sprites, collision
│   ├── gmParser.js      Bytecode → AST parsing
│   ├── gmSprite.js      Sprite parsing/rendering
│   ├── gmScene.js       Scene parsing (indexed pixel buffer)
│   ├── gmSound.js       Sound effects (Web Audio + SID emulation)
│   ├── gmMusic.js       Music playback (3-channel)
│   ├── gmDisk.js        D64 disk + localStorage persistence
│   ├── gmEditor.js      Instruction editor UI
│   ├── gmOpcodes.js     Opcode definitions
│   ├── gmCharset.js     C64 font rendering
│   ├── gmTools.js       Shared utilities
│   ├── d64lib.js        Pure D64/1541 disk parsing
│   ├── c64lib.js        C64 palette, PETSCII decoding
│   ├── c64Screen.js     320×200 canvas wrapper
│   ├── gif.js           GIF library (jnordberg/gif.js)
│   └── gifWorkerBlob.js GIF worker as inline blob (avoids file:// security restrictions)
├── css/
│   └── gm-ui.css        Master stylesheet (C64 colors, fonts, components)
├── tests/
│   ├── *.test.js        Vitest test files
│   ├── generate-golden-*.js  Golden file generators
│   ├── golden/          Golden snapshot files (JSON)
│   └── disks/           Test D64 disk images
├── tools/
│   ├── screenshot.js    Puppeteer screenshot utility
│   └── hex-viewer.html  Hex dump viewer
└── docs/
    ├── TODO.md          Future improvements
    └── references/      Reference materials
```

## Architecture Decisions

**Separate HTML pages (not SPA):** Each editor owns its state. Browser handles cleanup. Shared disk state via localStorage.

**AST execution with parent pointers:** Programs parsed into tree with `.parent` and `.parentIndex`. Jumps can target any label. No execution stack needed.

**Per-slot sprite instances:** 8 sprite slots, each gets own gmSprite instance. Critical for games where same sprite appears in multiple slots with different colors.

**Lazy palette resolution:** Sprites/scenes store C64 color indices (0-15), not RGB. Resolved at render time. Color changes just set dirty flag.

**Indexed pixel buffer:** Scenes use 160×200 Uint8Array of palette indices. Print/score writes to buffer. 16-30x faster than fillRect.

**320×200 native resolution:** All rendering at C64's true resolution. CSS scales to display. Simplifies coordinate math and collision detection.

## Data Architecture & Serialization

### Sources of Truth & Lifecycle

**LOADING (PRG → in-memory):**
1. PRG file (with binary data page) is the source of truth
2. `parseProgramData()` extracts: instructions (AST), mediaStore (JS objects)
3. After parsing, AST + mediaStore become the source of truth
4. Original PRG binary is no longer needed

**EDITING (in-memory):**
- AST: User edits instructions via editor UI
- mediaStore: User assigns sprites/sounds via editor UI
- Both are modified independently

**SAVING (in-memory → PRG):**
1. AST + mediaStore are the source of truth
2. `serializeProgram()` walks AST to find referenced media
3. Fresh binary data page is BUILT (output, not input)
4. Compact pointer table assigned (no gaps)
5. Result is a new PRG file

The editor maintains two sources of truth for program state:

1. **AST** (`currentProgramData.instructions`) - The instruction list, editable by the user
2. **Media Store** (`currentProgramData.mediaStore`) - JS array of media objects (gmSprite, gmSound, gmScene, gmMusic)

**IMPORTANT:** The JS property is `mediaStore`, NOT to be confused with "data page" which refers to raw binary data in PRG files. The mediaStore is a collection of JS objects, NOT raw binary data.

### Media Store Structure

```javascript
mediaStore[0] = null;  // Index 0 unused
mediaStore[1] = { name: "PLAYER", type: "sprite", sprite: gmSprite, spriteFileData: Uint8Array, ... }
mediaStore[2] = { name: "BOOM",   type: "sound", soundFileData: Uint8Array, ... }
mediaStore[3] = { name: "THEME",  type: "song",  songFileData: Uint8Array, ... }
// etc.
```

Instructions reference media by index: `sprite 0 is [1]` → use mediaStore[1].


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [needlestack/gmc64](https://github.com/needlestack/gmc64) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
