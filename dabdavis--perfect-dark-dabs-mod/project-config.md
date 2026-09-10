---
trigger: always_on
description: Fork of the [fgsfdsfgs/perfect_dark](https://github.com/fgsfdsfgs/perfect_dark) port.
---

# Perfect Dark port — simulant/mod fork

Fork of the [fgsfdsfgs/perfect_dark](https://github.com/fgsfdsfgs/perfect_dark) port.
`port` tracks upstream; work happens on `dabs-mod`.

`git checkout port` returns to stock at any time.

## What is written down here

Each note in `CLAUDE-notes/` is a thing that was got wrong once. Read the note
for an area **before** touching it — none of them are inferable from the code,
and each one cost a detour. This file keeps only what every session needs; the
notes are read when their area comes up.

- **The Windows build, wine, the pd.ini format** — [windows-build.md](CLAUDE-notes/windows-build.md): mingw prefix, WinHTTP, and why `Mod.LoadTextures=1` on its own line does nothing
- **chrs, bodies, heads, simulants, memory pools, mpconfig** — [chrs-and-memory.md](CLAUDE-notes/chrs-and-memory.md): a chr's prop is read before its tick; the ~50KB head copy that empties the stage pool; one head modeldef cannot sit on two bodies; ROM-resident structures never grow
- **Saves, eeprom, where pd.ini lives, the migration** — [save-format.md](CLAUDE-notes/save-format.md)
- **Menu text, `textMeasure()`, reaching the widescreen pillars** — [text-rendering.md](CLAUDE-notes/text-rendering.md)
- **Adding stages** — [stage-numbers.md](CLAUDE-notes/stage-numbers.md): `STAGE_IS_LEVEL()` admits 0x5e-0xff as well as the 27 free below the title; four ids are taken outside the table; the MP save format holds 7 bits
- **The Stage Loader: every mod's maps as arenas beside the mod loaded** — mods.md, "The Stage Loader": maps-only mounts never overlay; the registrar rescans on a swap; the branch's fixes (allocation, pool checks, room sizing, textures by stage); the importer splits a rebuilt texture table (30) and writes the `maps` block (31) so a mod's arenas come from its own tables under its own names — a file name is Perfect Dark's slot, not the map (GE-X's `crad` is Aztec)
- **Screenshots, the recorder, ffmpeg, GL capture** — [recording.md](CLAUDE-notes/recording.md): the frame is presented before `videoEndFrame()`; NV12 on the GPU; encoder detection; why it must never wait for the encoder; running on the real GPU with no window (llvmpipe hides driver limits)
- **Ghost Trials networking** — [ghost-trials.md](CLAUDE-notes/ghost-trials.md): WinHTTP and libcurl, why not one of them, and what the worker thread may touch
- **Check for Updates** — [updater.md](CLAUDE-notes/updater.md): `update.txt`, the baked-in channel, the two-rename swap
- **Mod directories, Load Mods, modconfig, `modcodediff`, the ROM symbol file, the data segment and importing a mod's weapon definitions** — [mods.md](CLAUDE-notes/mods.md): only the first mod dir joins the file search; files swap live, segments cannot; the `datasegment` block, `moddata.c`, and "where this stands" for continuing the import work
- **Texture packs** — [texture-packs.md](CLAUDE-notes/texture-packs.md): where a pack goes (one of four directories is read by nothing); decoding off the render thread and the backlog; the kept store and why a decode must never be handed over (textures flicker to the original otherwise); an emulator pack's image is the tile, the renderer maps the padded row; the *other* stretch is the game's own - a tile sampled past its edge, which no image edit can fix and **Stretched Edges** (`Video.StretchedEdges`: Original/Mirror/Repeat) can; PNG is ours, JPEG is stb_image, and the two row orders; font glyphs (the image is the whole tile, the outline pass wants both images); F7–F10; and **Community Packs**, which installs a pack from its author's release page - what is in the binary is the pack and not the release, the row-order marker is written at install time because v0.09 of the PD Plus pack renamed the `ext_tex` folder that used to say so, and the cover art is drawn through a stand-in tile the way an XBLA mesh's texture is (and is *not* turned over)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DabDavis/perfect-dark-dabs-mod](https://github.com/DabDavis/perfect-dark-dabs-mod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
