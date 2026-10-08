---
trigger: always_on
description: Dual-screen Android app (AYN Thor, Retroid Pocket Duo): an embedded libmgba core on the
---

# PokeDaisy — Project Notes for Claude

Dual-screen Android app (AYN Thor, Retroid Pocket Duo): an embedded libmgba core on the
top screen and a live companion for Gen 3 Pokémon games and ROM hacks on the bottom
screen. Read `README.md` for the user-facing feature list and `docs/DEVELOPMENT.md` for the
per-ROM support details, how each game was mapped, frontend setup and releasing; this file is
process context that isn't obvious from the code.

**This repo is public.** Never commit ROMs, saves or secrets. The SteamGridDB API key is
entered by the user at runtime and must stay that way. FireRed / Emerald art is rebuilt
from the player's own ROM (`RomArt`), not bundled. Keep it that way.

## Relationship to the FireRed QoL repo

PokeDaisy was split out of the user's **private** FireRed QoL ROM-hack repo (local
checkout `~/Projects/tests/my-rom-hacks`, where it lived as `tools/pokedaisey`
under the name pokedaisey) on 2026-10-06, without git history. That repo still owns:

- **The `gQolTelemetry` struct** the FireRed / Emerald QoL builds export
  (`include/qol_telemetry.h` via its `patches-firered/0009-*` / `patches-emerald/0001-*`,
  documented in its `docs/telemetry.md`). `companion/data/Telemetry.kt` decodes it byte
  for byte, so a struct change there needs the matching change here.
- **The Go `tools/telemetry-viewer`**. Several Kotlin files here are ports of its readers
  (`native.go`, `gen3mon.go`, `unbound.go`, ...) and say "keep in sync".
- **The pinned, built decomps** (`build/pokefirered`, `build/pokeemerald`, `build/pokeruby`,
  `build/pokehns`, ...) that the generator scripts and ui-preview read via `$DECOMPS`.

Public readers can't see that repo, so user-facing docs call it "the FireRed QoL project"
and don't link to it.

## Local setup on this machine

- `local.mk` (untracked, see `local.mk.example`): `DECOMPS` →
  `~/Projects/tests/my-rom-hacks/build`, `MON_ICONS` → that repo's
  `tools/telemetry-viewer/assets/pokemon`. `make` exports both; outside make, export them
  yourself for `python3 scripts/gen_*.py` or `gradle render`.
- `local.properties` (untracked): Android `sdk.dir`.
- `third_party/mgba`: git submodule at mGBA `0.10.5` (`make submodules`).
- ROMs/saves for headless captures: `scripts/host_roms.conf`. On-device paths:
  `scripts/roms.conf` (`/sdcard/Android/data/com.pokedaisey.app/files/...`).
- The package / applicationId was renamed from `com.pokedaisey.PokeDaiseyApp` to
  `com.pokedaisey.app`, so Android treats it as a new app. An old install's data (`roms/`,
  `saves/`, states, prefs) sits under the old id's `Android/data/` folder and doesn't
  carry over by itself.

## Headless captures

`native-capture/mgba_dump` (see its README) runs in the `pokedaisey-capture` Docker image
(`native-capture/Dockerfile`; the capture scripts build it on first use, or run
`make capture-image`). The image's `ENTRYPOINT` is `/bin/bash`, so always run
`docker run <image> -c "<full command>"`. Bare args make bash treat the command as a
script and fail with a misleading `cannot execute binary file`. Never reuse a `.ss`
savestate against a rebuilt ROM (stored code pointers go stale). Boot from a `.srm`
instead.

## UI work

**Look at every UI change before calling it done** — render it and read the PNG, don't
reason about Compose layout blind. Two ways, same screens:

1. **Paparazzi** (needs the Android SDK + Google Maven, i.e. the user's Mac):
   `./gradlew :app:recordPaparazziDebug --tests '*ScreenshotTest'`,
   then Read the PNGs in `app/src/test/snapshots/images/` (gitignored — a viewing tool,
   not a regression gate). Tests live in
   `app/src/test/kotlin/.../companion/ui/CompanionScreenshotTest.kt`: one per companion
   tab (via `CompanionScreen(initialTab = …)` — Paparazzi can't tap the tab bar) plus the
   shared `OptionSelector`/`OptionConfirm`, at the Thor bottom screen's 1240x1080 (landscape). Add a
   test when adding a screen. Paparazzi is pinned to **1.3.4** — the last release on
   Kotlin 1.9.24; newer ones need a Kotlin 2 bump first. Library/Settings (top screen)
   are private composables inside their Activities, so Paparazzi can't reach them; use
   the preview harness below for those.
2. **`ui-preview/`** (Maven Central only — works in cloud sessions
   where `dl.google.com` is blocked, which also blocks AGP, so the real app can't even
   compile there). A Compose **Desktop** project that compiles the real app sources
   (`companion/**`, `LibraryActivity`, `SettingsActivity`, …) against small hand-written
   Android shims and renders PNGs of every companion tab _and_ the top-screen
   Library/Settings, including tapped states (selectors, confirms, sub-pages):
   `cd ui-preview && gradle render` → `build/shots/*.png`
   (`-Ponly=settings` filters by name, `-Pgame=EMERALD` switches game). It's also the
   only compile check available in such sessions, so run it after any UI edit there. See
   its README for the shims and quirks (CMP 1.5 / Kotlin 1.9.22, `Icons.AutoMirrored`
   rewritten to `Icons.Filled`, a stale incremental cache once faked "Unresolved
   reference" errors — delete `build/kotlin` if errors look impossible).

**The app's one look is FireRed's OPTION screen** (`companion/ui/GbaMenu.kt`): white

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lidor30/pokedaisy](https://github.com/lidor30/pokedaisy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
