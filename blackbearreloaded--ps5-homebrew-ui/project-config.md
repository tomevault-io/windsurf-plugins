---
trigger: always_on
description: This file is for coding agents (and people in a hurry). It says what this
---

# Instructions for AI agents

This file is for coding agents (and people in a hurry). It says what this
repository is, where to look, how to work in it, and what "done" means here.
The detailed guides are linked where you need them; read those instead of
guessing.

## What this is

A reference for building console-grade user interfaces for PS5 homebrew with
OpenGL 4.6, and one native app ("Homebrew UI Lab") that shows all of it.

<!-- BEGIN:counts -->
Right now: **21 designs**, **30 themes**, **99 components**.
<!-- END:counts -->

It has four levels. Enter at the highest one that does the job:

| Level | What it gives you | Where | Guide |
| --- | --- | --- | --- |
| **Components** | Whole pieces that own their focus, values, motion and sound: lists, grids, tabs, menus, dialogs, notifications, forms, pickers, a keyboard, tables, charts, media controls, HUD pieces, layout and focus navigation | `src/ui/components/` | [docs/COMPONENTS.md](docs/COMPONENTS.md), [docs/COMPONENT_INDEX.md](docs/COMPONENT_INDEX.md) |
| **Widgets and themes** | `ui::Painter`: stateless themed controls; `ui::Theme`: a design language as data | `src/ui/widgets.hpp`, `src/ui/theme.cpp` | [docs/THEMES.md](docs/THEMES.md) |
| **The kit** | Shapes, text, images, glass, backdrops, springs, input, the mixer and its cue vocabulary | `src/gfx`, `src/ui`, `src/core`, `src/audio` | [docs/KIT.md](docs/KIT.md) |
| **Designs** | Complete screens, one file each, switched with L1/R1 in the app | `src/concepts/*.cpp` | [docs/DESIGNS.md](docs/DESIGNS.md) |

Everything on screen is drawn through OpenGL by the kit. Do not add another
rendering path.

## If the task is...

| Task | Do this |
| --- | --- |
| "Build me a settings screen / library / store / menu" | Start a design ([docs/BUILDING_A_DESIGN.md](docs/BUILDING_A_DESIGN.md)) and assemble it from components. Find them in [docs/COMPONENT_INDEX.md](docs/COMPONENT_INDEX.md). `src/concepts/components/*_page.cpp` are worked examples, one per group. |
| "I need a widget that does X" | Search [docs/COMPONENT_INDEX.md](docs/COMPONENT_INDEX.md), open the header (it starts with a usage example), then the group's guide in `docs/components/` for every knob, slot, event and cue. |
| "Make it look like Y" | Pick or write a `ui::Theme` ([docs/THEMES.md](docs/THEMES.md)); assign it to `component.style.theme`. Then turn the component's own knobs. |
| "Move the focus between things" | By hand, by edge exits, or with `ui::FocusGroup`: [docs/COMPONENTS.md](docs/COMPONENTS.md), [docs/components/layout.md](docs/components/layout.md). |
| "Add a new component" | The checklist under *Adding things* below. |
| "Use this in my own app" | [docs/ADOPTING.md](docs/ADOPTING.md): copy `src/gfx`, `src/ui`, `src/core`, `src/audio` and the assets. |
| "Why is it slow / is it fast enough" | [docs/PERFORMANCE.md](docs/PERFORMANCE.md) (rules and numbers measured on a console). |
| "Run it on the console" | [docs/CONSOLE_VALIDATION.md](docs/CONSOLE_VALIDATION.md) and *Console work* below. Ask the owner first. |

Other guides: [docs/CRAFT.md](docs/CRAFT.md) (the quality bar and its
checklist: read it before any UI work), [docs/SOUND.md](docs/SOUND.md),
[docs/BACKDROPS.md](docs/BACKDROPS.md),
[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md),
[docs/platform/](docs/platform/) (building, packaging, deploying).

## Repository map

```
src/ui/components/        the component library (one header + source per component)
src/ui/components.hpp     includes every component
src/ui/                   fonts, glyphs, motion helpers, themes, Painter widgets
src/gfx/                  draw list, GL batch, fonts, backdrops, renderer
src/audio/, src/core/     mixer, cues, music; input, springs, settings, save files
src/concepts/             the designs; components/ holds the gallery's pages
src/app/                  the shell (L1/R1 switch), the tour, the design interface
src/platform/ps5/         display, controller, audio output, system services
host/                     PC renderer (Mesa): pictures, clips, manifest
tests/unit/               GoogleTest suite, no OpenGL needed
tools/                    build, snapshots, media, docs, console validation
assets/                   baked fonts, two sound-effect sets, music
sce_sys/                  what the PS5 home screen shows: icon, backgrounds, music, param.json
docs/                     the guides; docs/media is generated
```

## The loop

Work on a PC first. A console is only for the final check.

```bash
tools/host-snapshots.sh build/snapshots <design id>   # build for the PC, run the tour, write PNGs
make test-unit                                        # GoogleTest suite, sanitizers on
make                                                  # PS5 app folder in dist/
make lint                                             # format, static analysis, metadata
```

1. Change code.
2. Render the design and **look at the pictures** in `build/snapshots/`. A
   build that compiles proves nothing about a UI. Crop and enlarge anything
   you are unsure about.
3. Run the tests.
4. Build for the PS5.

## Local preview: see the UI without a console

The app's interface builds for a PC and renders off-screen through Mesa's
software renderer. It is the same UI code and the same shaders as on the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blackbearreloaded/ps5-homebrew-ui](https://github.com/blackbearreloaded/ps5-homebrew-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
