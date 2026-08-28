---
trigger: always_on
description: Onboarding for anyone new to this repo, human or AI. `README.md` is for people
---

# AGENTS.md

Onboarding for anyone new to this repo, human or AI. `README.md` is for people
installing it, `DEVELOPMENT.md` covers the toolchain and build. This file is the
working reference: what is known about the hardware, what is known to be broken,
and the rules that are not obvious from reading the code.

## What this is

Guppyscreen is a touch UI for Klipper printers, talking to Moonraker over a
websocket, built on LVGL as a standalone binary with no X or Wayland underneath.
It draws straight to `/dev/fb0` and reads touch from `/dev/input/event0`.

This repo is a **takeover of an abandoned project**. Upstream
`ballaswag/guppyscreen` stopped at commit `07409cb` on 2024-07-15. We forked
from that commit and are picking it back up.

It is still stopped, re-checked 2026-08-18: `main` is our fork point exactly,
and the `dev` and `btt_pad7` branches are stale from December 2023 with nothing
in them that is not already in `main`. What is left there is 63 open issues and
6 open pull requests, all triaged in `docs/upstream-issues.md`. Read that before
acting on anything anyone reports upstream, because about half of it we have
already fixed.

Our target is a **Creality K1 Max**. Everything measured about it is in
`docs/k1max-facts.md`. Do not guess at hardware details, that file has the real
values.

## Read these first

| File | What it is for |
| --- | --- |
| `docs/k1max-facts.md` | Real hardware, firmware, framebuffer and input values from the printer |
| `docs/audit.md` | Known bugs, inherited and our own, with severity and suggested order |
| `docs/upstream-issues.md` | Upstream's 63 open issues and 6 open PRs, triaged against our tree |
| `DEVELOPMENT.md` | Toolchain, build targets and running the simulator |

## Build

Three targets, all through one script.

```sh
scripts/setup-toolchain.sh          # once, downloads both cross toolchains
scripts/build.sh mips               # K1 / K1 Max binary
scripts/build.sh arm                # aarch64, Raspberry Pi and BTT Pad
scripts/build.sh sim                # x86_64 SDL build that runs on your desktop
```

Useful variants:

```sh
scripts/build.sh mips zbolt         # Z-Bolt icon set instead of Material
scripts/build.sh mips --small       # Ender 3 V3 KE / Nebula Pad sized panel
scripts/build.sh mips --clean       # rebuild the vendored libs too
PRINTER_HOST=192.168.1.202 scripts/build.sh sim   # point a *new* sim config at a printer
```

Output is `build/bin/guppyscreen` for every target. Two stamp files track what
was last built so nothing stale gets linked: `.vendor-target` for the
architecture, which is all libhv, spdlog and libwpa_client care about, and
`.build-flags` for architecture plus theme plus small screen, since those change
`-D` defines that affect every object of ours.

## CI

One workflow, `.github/workflows/build.yml`. It builds the same four variants
CI has always built, plus the simulator, and it **calls `scripts/build.sh`**
rather than repeating the flags. If you add a build option, put it in the script
and reference it from the matrix, so the two cannot drift.

It runs on push to `main` and on pull requests. There is no tag trigger:
publishing happens per push, so tagging is not how releases are made here.

Two version shapes, and `update.sh` keys off them:

| Trigger | Version | Published as |
| --- | --- | --- |
| local `scripts/build.sh` | `dev-<sha>` | nothing, never published |
| pull request | `dev-<sha>` | nothing, never published |
| push to `main` | `<date>-<sha>` | its own release, tagged the same |

Every push that changes source and builds cleanly gets its own release, so the
release list is a build history. Do not go back to a single fixed tag that
overwrites itself: that was the old arrangement and it left nothing to roll
back to.

**Documentation-only pushes still build but do not publish.** The `version` job
diffs against `github.event.before` and skips the release when every changed
path matches `*.md`, `docs/`, `screenshots/` or `LICENSE`. The filter lists what
to ignore rather than what to build, so an unrecognised path errs towards
publishing. Remember that `installer.sh`, `update.sh`, `k1/` and `themes/` ship
inside the tarball and count as source even though nothing compiles them.

If it ever skips a release you wanted, run the workflow manually: a
`workflow_dispatch` always publishes.

The version is worked out once in the `version` job and passed to the build and
release jobs. Computing it in both races across UTC midnight and would tag a
release differently from the string compiled into the binary. The tag, the
release name and `.version` are all the same string.

Releases are normal releases, not prereleases, so `releases/latest` resolves to
the newest build. `update.sh` and both installers follow that rather than any
fixed tag.

The release is published from its own job, after the whole matrix **and** the
simulator build pass. Do not move it back into the matrix: four parallel jobs
race to create the same tag, and a broken variant would produce a half
populated release.

Release notes are generated in the workflow. Keep them practical: which asset
suits which printer, and how to install. Anyone arriving from the original

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JuggyMcNutty/vibescreen](https://github.com/JuggyMcNutty/vibescreen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
