---
trigger: always_on
description: A ZX Spectrum emulator for Android, built on an unmodified Fuse core. `README.md` is for people using the app and nothing else — keep build, test and release material out of it. `docs/USING.md` is the tour of the controls — every quick bar row, the ☰ sheet, the hotkeys — and is where a how-to that outgrows a README line goes; its icons are generated from `res/drawable` by `scripts/icons-to-svg.py`, so an icon changed in the app and not there leaves the guide illustrating a button that no longer 
---

# Zedex — working notes

A ZX Spectrum emulator for Android, built on an unmodified Fuse core. `README.md` is for people using the app and nothing else — keep build, test and release material out of it. `docs/USING.md` is the tour of the controls — every quick bar row, the ☰ sheet, the hotkeys — and is where a how-to that outgrows a README line goes; its icons are generated from `res/drawable` by `scripts/icons-to-svg.py`, so an icon changed in the app and not there leaves the guide illustrating a button that no longer looks like that, silently. `docs/DEVELOPING.md` covers building, driving it from adb, the tests and releases; `docs/INTERNALS.md` how the core is wired in. This file is the operational knowledge that is easy to get wrong and expensive to rediscover.

## Hard rules

- **Never modify `vendor/`.** The Android backend is swapped in at link time (`--with-fb`, skip `ui/fb`, link `native/ui/android`, weaken two symbols with `llvm-objcopy`) — reaching for a patch usually means looking in the wrong layer.
- **A Fuse change is a patch in git, never an edit in place.** The build compiles a copy with `native/patches/*.patch` applied (`scripts/fuse-src.sh`, in `build-native/src`, a git repo tagged `upstream` at the pristine tarball) — `fuse-src.sh save` turns commits into the series, `reset` proves a fresh clone gets the same tree. A build tree remembers the source dir it was configured against, so pointing it at the patched tree while still configured for `vendor/` links a library with none of the patches, silently; the baseline is the tarball, not `vendor/`, since Fuse's codegen writes `settings.c` into the source dir. See *Patching Fuse* in `docs/DEVELOPING.md`.
- **The upstream version is pinned and stays pinned** (`FUSE_VER`, `LIBSPECTRUM_VER` in `scripts/build-native.sh`, SHA-256 checked). Bumping needs the version, the hash, *and* `build-native.sh clean` — `fetch` skips a version already extracted, so a bumped number over an old `vendor/` builds the old code and says nothing.
- **The debug build is a package of its own**, `dev.ldlab.zedex.debug` — `appops`/`run-as`/`am start` need the right one. Fuse finds its data relative to `argv[0]` now, so the `PKG` baked into `FUSEDATADIR` is only a fallback nothing reaches.
- **Only swallow a key Fuse can use.** Consuming the volume keys so Fuse could ignore them is how the phone's volume buttons stopped working; `FuseNative.mapsKey()` checks Fuse's own keysym table and passes unknowns to `super`.
- **The version lives only in `version.properties`.** `versionCode` is `major*10000 + minor*100 + patch`; the release workflow checks the tag against it, so `git tag v1.2.3` on `version=1.2.2` fails before it builds.
- **`app/debug.keystore` is committed on purpose** — Gradle's own debug key is per machine, so no CI build could update a local one, or another CI build's.
- **`targetSdk` is Play's floor, and it moves** (36 since August 2027; 35 before that). It is independent of `minSdk`, which stays at 30 — the target says which behaviours the app has been tested against, not who may install it, and Android 11 and 12 are still supported. Targeting 35+ needs **16 KB page aligned** native libs (`-Wl,-z,max-page-size=16384`): the NDK's CMake toolchain sets it, but a hand-driven cross-compile gets the 4 KB default, which runs everywhere and is unmappable on a 16 KB device — the script asserts `0x4000`. Bumping `targetSdk` means re-running the native build, not just Gradle. Targeting 36 additionally turns **predictive back on by default**, so an activity that handled `KEYCODE_BACK` in `onKeyDown` silently stops being asked — both that claim it register a dispatcher callback from API 33 and keep the old path for 30–32 — and makes Android **ignore orientation, resizability and aspect-ratio requests on displays 600dp and wider**, which costs nothing here because the app asks for none of them.
- **Everything of ours stays inside the safe area.** API 35 lays a window into the display cutout unconditionally (letterbox mode reads `ALWAYS`), and hiding the system bars does not help — a hidden bar reports a zero inset but `displayCutout()` does not.
  - `EmulatorLayout` keeps a `safe` rect: `arrange()` gets the window minus it, `placeChild` adds the offset back — no other code needs to know a cutout exists.
  - Everything else calls `SafeArea.fit()` on `android.R.id.content`.
  - Test with a cutout — AVDs have none: `adb shell cmd overlay enable --user 0 com.android.internal.display.cutout.emulation.hole`, then reboot; `dumpsys window | grep DisplayCutout` should show a top inset.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dimitriuz/zedex](https://github.com/dimitriuz/zedex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
