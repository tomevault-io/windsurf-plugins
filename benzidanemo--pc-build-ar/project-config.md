---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A rebuild of the 2007 Cisco IT Essentials "Virtual Desktop" PC-assembly simulator (Flash 8 / ActionScript 2) as an offline React app. Algerian high-school CS teachers use it to teach students how to put a PC together. The UI is Arabic (RTL), with French and English hardware terms shown alongside. It ships as a Windows app built with Pake, and `dist/` must also work opened straight from a folder (`file://`) as a fallback for lab PCs without WebView2 (Windows 7). That's why `base: './'` is set, with no CDN and a self-hosted font. `vite.config.ts` also builds one classic IIFE script (no `type="module"`, no `crossorigin`; browsers block both from `file://`) plus one CSS file, targeting Chrome 109 / Firefox 115, the last browsers on Windows 7. Lesson data is an eager glob inside that script. Target hardware is mouse-and-keyboard desktop lab PCs, not touch screens.

## Commands

```bash
npm run dev                       # Vite dev server (use --host 127.0.0.1 if localhost hangs)
npm run build                     # tsc -b && vite build -> dist/
npm run package [-- x64|x86]      # build + Pake (GNU toolchain), 64- and 32-bit -> release/ (msi, portable zip, web.zip, guide)
npm test                          # vitest: auto-solves every lesson against the real extracted data
npx vitest run -t "lesson 2"      # one lesson
node tools/smoke.mjs <lesson> [url]   # plays a lesson through the real UI in headless system Chrome; screenshots -> tools/.cache/smoke/lesson<N>/; exits 1 on console errors
node tools/smoke.mjs 6 "file:///$PWD/dist/index.html"   # the web.zip case: run before packaging (blocked file:// resources show up as console errors)
HINTS=off node tools/smoke.mjs 2 [url]  # same with "Show instructions" unchecked
node tools/smoke.mjs test [url]       # TEST mode: all 7 stages then the results screen (~25 min)
CPU=4 node tools/smoke.mjs 6 [url]    # slow-PC check: logs load time and fps during an assembly animation
CDP=http://127.0.0.1:9222 node tools/smoke.mjs 1   # drive the packaged app (see Packaging)
python tools/contact_sheet.py tools/.cache/smoke/lesson<N>   # tiles those screenshots into sheet.png
```

Asset pipeline (needs Python + Pillow and JDK 21 at `C:\Program Files\Java\jdk-21`; JPEXS is unpacked in `tools/.cache/ffdec/`):

```bash
J="/c/Program Files/Java/jdk-21/bin/java"; FF=tools/.cache/ffdec/ffdec-cli.jar
"$J" -jar $FF -format image:png_gif_jpeg -export image tools/.cache/images/<Lesson> legacy/models/<Lesson>.swf
"$J" -jar $FF -format shape:svg -export shape tools/.cache/shapes/<Lesson> legacy/models/<Lesson>.swf
"$J" -jar $FF -export script tools/.cache/scripts/<Lesson> legacy/models/<Lesson>.swf   # decompiled AS2, the behaviour reference
python tools/extract_lessons.py [Lesson]   # SWF -> tools/.cache/manifests/<Lesson>.json
python tools/build_assets.py [Lesson]      # -> public/lessons/<Lesson>/ + src/content/lessons/<Lesson>.json (parallel, cached)
# «اكتشف القطع» photos + callouts from legacy/media/explore/*.swf (export images and texts first, see the script's docstring):
python tools/extract_explore.py [--debug]  # -> public/media/explore/ + src/content/explore.json; --debug outlines callouts in tools/.cache/explore-debug/
# today's parts in «اكتشف القطع»: photos from Wikimedia Commons
python tools/fetch_photos.py search "<query>"       # Commons files with their licence (OK = allowed)
python tools/fetch_photos.py preview "File:..."     # small previews -> tools/.cache/photo-candidates/
python tools/fetch_photos.py                        # fetch tools/photos.json -> public/media/explore/modern/ + photoCredits.json + CREDITS.md
```

Transcription helpers:
- `python tools/summarize_lesson.py <Lesson>` prints clip labels, clickable children with frame ranges and rects, and the non-boilerplate script lines per frame.
- `python tools/render_frames.py <Lesson> <clip> <frames…>` renders what specific frames show, with named rects outlined.

`tools/convert_xml.py` generated `src/content/parts.json` once. It is now hand-maintained, so don't regenerate it over edits.

## Packaging

`tools/package.mjs` runs the project-local `pake-cli` on `dist/` once per arch, each in `release/.build/<arch>/`, because Pake writes `<name>.msi/.exe` to its cwd. It uses `--use-local-file` and `--windows-toolchain gnu`. It then assembles `release/`: `PCBuilderDZ_<v>_{x64,x86}.msi`, `PCBuilderDZ_<v>_portable_{x64,x86}.zip`, `PCBuilderDZ_<v>_web.zip` (dist/, for Windows 7 without WebView2), and `دليل-الأستاذ.md`.
- **Toolchains:** x64 uses MSYS2 `ucrt64` gcc. x86 uses MSYS2 `mingw32` gcc (`pacman -S mingw-w64-i686-gcc`) plus `rustup target add i686-pc-windows-gnu`. The x86 linker, CC, AR and WINDRES are set through `CARGO_TARGET_I686_…` / `*_i686_pc_windows_gnu`. The 64-bit gcc stays first on PATH for the host build scripts. Everything is set per process; `MSYS2_ROOT` overrides `C:\msys64`. Pake sets `RUSTUP_TOOLCHAIN=stable-x86_64-pc-windows-gnu` itself.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BenzidaneMo/pc-build-ar](https://github.com/BenzidaneMo/pc-build-ar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
