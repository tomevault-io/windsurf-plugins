---
trigger: always_on
description: This repo holds one test app per console, each in its own subdirectory.
---

# CLAUDE.md — joypad-tester

This repo holds one test app per console, each in its own subdirectory.
The patterns below describe how each console subdir is structured so
that new consoles can be added without churning the top-level scaffolding
or the CI/release wiring.

## Repository shape

```
.
├── CLAUDE.md            # this file
├── LICENSE.md           # MIT, covers repo scaffolding + any future common/
├── README.md            # top-level overview
├── .github/
│   ├── FUNDING.yml      # Sponsor button → RobertDaleSmith
│   └── workflows/
│       ├── verify-build.yml   # matrix CI for every push/PR
│       └── release.yml        # tag-driven per-console releases
├── <console>/           # one subdir per supported console
│   ├── VERSION          # bare semver string, e.g. "0.1.0"
│   ├── CHANGELOG.md     # console-scoped, header style: "## v0.1.0 — YYYY-MM-DD"
│   ├── LICENSE.md       # whatever the upstream code's licence is
│   ├── README.md        # console-scoped overview
│   ├── Makefile         # console-specific build
│   ├── source/ (or ppc/, etc.)
│   ├── build*/          # build outputs; intermediate files gitignored,
│   │                    # final artifacts (.dol/.gba/_payload.c) committed
│   │                    # if downstream consumers need them prebuilt
│   └── ...
└── common/              # (future) cross-console shared helpers
```

Subdir names are short 3-letter codenames matching homebrew-community
usage (`gcn`, `gba`, `pce`, future `n64` / `snes` / …). The same
codename is used as the release-tag prefix (`<codename>-v<semver>`).
Don't introduce full-localized-name subdirs (`gamecube/`,
`gameboyadvance/`, etc.) — the codename is the source of truth. Also
avoid 2-letter codenames (`gc`) since they tend to collide with
unrelated paths in vendored / sibling repos (joypad-os has its own
`gc/` subdir for the GameCube host that has nothing to do with our
subdir codename).

Existing console subdirs at time of writing:

- `gcn/` — GameCube + Wii test app (libogc2-based, builds .dol; also
  surfaces N64 controllers via the passive adapter on GC ports).
  Origin: zlib (corenting GC-Controller-Test); `gcn/LICENSE.md`.
- `gba/` — GBA multiboot payload, two variants from one source tree
  (eyes for joypad-os consumers, tester for the GameCube host or
  flashcart use). Origin: MIT (Doridian Joybus-PIO); `gba/LICENSE.md`.
- `pce/` — PC Engine / TurboGrafx-16 test app (HuC, builds .pce).
  Detects 2-button / 6-button pads and the PCE mouse via the standard
  joypad port + multitap. Origin: MIT (dshadoff PCE_Mouse_Test);
  `pce/LICENSE.md`.
- `3do/` — 3DO Opera test app (trapexit/3do-devkit-based, builds
  iso9660 .iso). Reads the daisy-chain control pad and renders live
  button state. Origin: MIT (original src/main.cpp) + ISC for the
  vendored devkit helpers; `3do/LICENSE.md`. Toolchain is Linux-only
  x86 binaries running under `--platform=linux/amd64` in the build
  container, so Docker is mandatory for this subdir.

## Console subdir conventions

Each console subdir is a self-contained product. Adding a new one
should not require touching anything outside its own directory, with
the exception of the two CI workflows and CLAUDE.md's "Existing console
subdirs" list.

### Files every console subdir must have

| File          | Purpose                                                       |
|---------------|---------------------------------------------------------------|
| `VERSION`     | Bare semver string. Must match the release tag (see below).   |
| `CHANGELOG.md`| Per-version release notes — the canonical source of truth for what shipped. One section per release, header `## v<semver> — <date>`, body = 1-paragraph summary + a small Highlights list. The GitHub Release body is a one-line pointer to this file (GitHub's auto-rendered Assets list handles the file list, no need to repeat it in either the changelog or the release body). |
| `LICENSE.md`  | Whatever the upstream code's licence is (zlib, MIT, …).        |
| `README.md`   | Audience-facing overview: what the app is, how to build, how to embed. The long-form feature breakdown lives here, not in the changelog. |
| `Makefile`    | Build entrypoint. Use a Docker-based toolchain if it eases CI. |

### README structure (every console subdir)

Each `<console>/README.md` follows the same top-to-bottom outline so
they're navigable as a set. Sections marked *(optional)* are included
only when the console has that surface; everything else is required.

1. **Title** — `# Joypad Tester — <Console>`
2. **Intro** — one paragraph: what this build does and a link back to
   the top-level repo (`[Joypad Tester](../README.md)`).
3. **`## What it tests`** — concrete ASCII example of the on-screen
   output, then a table covering the per-port / per-variant matrix
   (controller types, payload variants, etc.). Always state which
   fields are unavailable on this platform so reading "zeros" isn't
   ambiguous.
4. **`## (feature deep-dives)`** — one `##` section per non-obvious
   subsystem (accessory paks, alt protocols, BIOS quirks, idle modes,
   etc.). Use as many as needed. Keep them concrete: protocol bytes,
   timings, register addresses where relevant.
5. **`## Build`** — preferred toolchain commands (devkitPro / devkitARM

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joypad-ai/joypad-tester](https://github.com/joypad-ai/joypad-tester) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
