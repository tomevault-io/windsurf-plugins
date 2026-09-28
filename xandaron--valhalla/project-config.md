---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Valhalla is a 3D graphics engine written in Odin against Vulkan 1.4, intended to grow into a full
game. Graphics is one component of that game, not the whole project.

## House rules

### The engine is cross platform

Valhalla targets more than Windows. Do not reach for a platform API because it is the quickest
way to solve something. If a problem genuinely has no portable solution, every supported target
needs a real implementation — a Windows path plus empty stubs is not acceptable. Where a
platform needs nothing, say why in a comment so the empty body reads as a conclusion rather
than a gap.

Platform code goes in its own file using Odin's filename suffixes rather than `when ODIN_OS`
blocks: `Graphics_windows.odin` and `Graphics_darwin.odin` are selected implicitly, and
`Graphics_unix.odin` carries an explicit `#+build linux, freebsd, openbsd, netbsd`. Each defines
the same procedure, so exactly one exists per target.

`odin check src -target:<target>` is the check. It currently stops on `vendor:stb` missing
prebuilt non-Windows binaries, which is a toolchain gap rather than a code problem; to verify
platform files in isolation, copy them to a scratch package outside the repo with a stub `main`
and check that against each target. That also catches a missing or duplicated definition, which
is the main hazard of this layout.

### One file per component — do not split code up

Do **not** create new `.odin` files. When you add functionality, put it in the file that already
owns that component. `src/Graphics.odin` is large on purpose; size alone is never a reason to
split it.

If something genuinely warrants its own file, say so and wait — that call is the user's, not
yours. This has already been reversed once: the memory allocator, staging ring and descriptor
heap were each written as separate files and later folded back into `src/Graphics.odin`.

The sanctioned exception is platform code, which uses Odin's filename build tags
(`Graphics_windows.odin`, `Graphics_darwin.odin`, `Graphics_unix.odin`). Do not fold these back
into `Graphics.odin`.

Inside a file, organise with section banners instead:

```odin
// ===[ Device Memory ]=========================================================
```

`grep "^// ===\[" src/Graphics.odin` lists the sections. Add new code to the section it belongs
to; add a new section only if it is genuinely a new area of responsibility.

### Do not write comments

Do not write comments. Not explanatory ones, not "why" ones, not file headers. The code stands on
its own.

The only exception is the section banners described above, which are structure rather than prose.

## Task tracking

`README.md` holds the roadmap and task list. The roadmap describes large undertakings; the tasks
section breaks each one into checkable items. Keep both current: tick items off as they land, and
add new ones there rather than leaving TODO comments scattered in the source.

## Commands

```sh
odin build src --debug --linker:radlink -out:bin/valhalla.exe     # build
./bin/valhalla.exe ./demo/demo.project   # run (argument is the project file)
./bin/valhalla.exe ./demo/demo.project ./scenes/bench.scene   # optional scene override
```

There is no test suite, linter or build script. `odin build` is the only check; treat a clean
build plus a clean validation run as the bar.

### Running it for verification

The app is a GUI program, so a change is not verified until it has been run. Two things matter:

- Kill it and you skip `cleanupGraphics`, which is where the pipeline cache is saved, leak
  reporting runs, and teardown validation errors surface. Close the window instead
  (`Process.CloseMainWindow()` from PowerShell), or press **Ctrl+Q**, which is a clean exit that
  works even with an imgui text field focused, so shutdown actually executes.
- Validation output goes to **stderr**, ordinary logging to stdout. Check both.
- **Never drive the real mouse or keyboard** (`SetCursorPos`, `mouse_event`, `SendKeys`), and
  never minimise, close or raise the user's windows. The user is working on the same machine;
  simulated input has landed in their windows before. Use the external API instead.

### The external API

`-external[:port]` (default 47470) makes the app listen on `127.0.0.1` for newline-terminated
text commands, one reply line each (`ok ...` or `err ...`). It lives in `src/External.odin`.
The window is shown without taking focus in this mode. Input is injected through `handleInput`,
the same path GLFW's callbacks use, so it exercises imgui and picking exactly as real input does.
A click spans three frames (move, press, release) so imgui's hover state is current when it lands.

| Command | Effect |
| --- | --- |
| `ping` | `ok pong` |
| `state` | JSON: scene, loaded scenes, UI hidden, fly mode, selection, inspector tabs, window size, FPS |
| `move X Y` / `click X Y [left\|right\|middle] [ctrl] [shift] [alt]` / `down` / `up` | mouse, in window coordinates (the screenshot's pixels at 100% scaling) |
| `scroll DY`, `key NAME [mods]`, `text STRING` | wheel, key press+release (`F1`, `ENTER`, `A`, ...), typed characters |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xandaron/Valhalla](https://github.com/xandaron/Valhalla) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
