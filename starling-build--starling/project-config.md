---
trigger: always_on
description: The Starling desktop: Swift shell + compositor + framework + apps, running on
---

# starling-desktop — project guide

The Starling desktop: Swift shell + compositor + framework + apps, running on
the Flutter engine's C core from the sibling repo **starling-engine**
(`../starling-engine`, reached via the repo-root `engine` symlink — run
`./bootstrap.sh` after cloning). Read its `CLAUDE.md` too: a feature usually
touches both repos and both get committed.

Everything needed to build, run, drive, and package the desktop is in this repo
(`build/`). The older `starling-os` repo still holds the Bazel-built Starling OS
image and its QEMU boot gates; the desktop dev loop no longer depends on it.

## Layout

```
sdk/       Flutter→Swift framework port (SwiftPM package "FlutterSwift", no Dart VM)
registry/  the app registry — the ONE description of every app the desktop knows
           about (catalog.d/*.app), shared by the shell and the App Store
shell/     DesktopShellApp package — its own CLAUDE.md in Sources/DesktopShellApp/
  Sources/DesktopShellApp/   Shell/ Window/ Compositor/ Wayland/ Taskbar/ Launcher/ Portal/ Utils/
  Sources/WaylandServer/     the Wayland compositor, in C (~5k lines)
  Sources/PortalService/     xdg-desktop-portal (sd-bus/basu)
  Sources/X11Server/         in-tree X server (DRI3/Present) — do not touch unless asked
apps/      first-party apps, one SwiftPM package each
build/     stage.sh (assembles the tree — the single definition of the layout),
           run-desktop.sh (run it), shell-drive.py (input + screenshots),
           package-desktop.sh (Ubuntu .deb), app-run/app-install,
           vendored flutter_assets, bundled wallpapers live in shell/Resources
```

## Build & iterate

- Shell/app Swift change → `cd shell && swift build -c release` (apps likewise),
  then `build/run-desktop.sh` (it re-stages first).
- Engine C++ change → rebuild in the **engine repo** (`ninja -C engine/src/out/host_debug
  libflutter_linux_drm.so libflutter_engine.so`) — no shell relink needed.
  Rebuild host_release too before packaging.
- Run: `build/run-desktop.sh` — stages into `.stage/` then runs from there.
  Drive/screenshot with `sudo build/shell-drive.py …`.
- Package: `build/package-desktop.sh` → .deb. It consumes `build/stage.sh`, which
  is the **single definition of the layout** — change assembly there, never in
  the packager alone.
- Test: `test/run.sh` (~0.4s, no GPU — run it on every change),
  `test/run.sh --build` to compile everything and the .deb,
  `sudo test/run.sh --functional` to drive a live desktop, and `test/vm.sh`
  for the release gate (.deb on a clean VM through a real GDM login — the only
  tier that can see privilege-path bugs). See `test/README.md`.
- Building on a machine with **nothing installed** (both repos, toolchains,
  apt packages, `gclient`) → `docs/BUILDING.md`.

**Ubuntu 26.04 LTS is the base platform**, for dev, test, and the shipped .deb.
The 6.2.4 toolchain is an ubuntu24.04 build, so 26.04 needs two fixes — both
already in-tree, so `swift build` takes no special flags: `bootstrap.sh` adds
the toolchain's `libxml2.so.2` symlink, and every `Package.swift` carries
`glibcMathCompat` (`-D_GLIBCXX_MATH_H` + a force-included glibc `math.h`) for
glibc 2.43's `<cmath>` clash. Delete both when swift.org ships a 26.04
toolchain. Why, in full: `docs/BUILDING.md`.

**Always run from the staged tree, never straight out of `.build`.** Child apps
are spawned with `LD_LIBRARY_PATH` scrubbed (`STARLING_CHILD_HOST_GL`) and
resolve libraries through their own `$ORIGIN`/RUNPATH only, so they work solely
when the libraries sit beside them — exactly what staging (and the package)
arranges. A `.build`-relative layout appears to work right up until a child app
dies with `libflutter_engine.so: cannot open shared object file`.

## Apps are data, not code

Adding an app is **one file**: `registry/catalog.d/<id>.app` (plus a launch
recipe in `build/app-run.sh` and an install recipe in `build/app-install.sh`
if it is a third-party host app). Never add an app id to a table in the shell
or the store — there are no such tables any more, and reintroducing one is how
this drifted the first time: an app was in seven tables and missing from two,
so it launched but had no dock icon and no real icon.

- `registry/catalog.d/*.app` — shipped, read-only. Name, tile colour, glyph,
  dock position, store copy, install/launch recipe names, the `.desktop`
  entries to read, the window classes its windows report. Read by the shell's
  launcher and dock, the App Store, and `app-install`.
- `/var/lib/starling/installed.d/<id>.app` — written by `app-install` on a
  successful install, deleted on removal. Carries what only exists once the
  app is on disk: its `.desktop` file, `StartupWMClass`, icon, version. The
  shell watches this directory (inotify), so an install lights up the launcher
  and dock with no relogin.
- Debug it with `app-install --record <id>`, which re-resolves and rewrites one
  record without installing anything (`STARLING_APP_RECORDS=<dir>` to test
  unprivileged), then `cat` the result.

**Window → app identity is `app_id`, never the title.** A window's
`xdg_toplevel.set_app_id` matches the `StartupWMClass` in the app's `.desktop`
entry; that pairing exists for exactly this purpose and `app-install` records

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [starling-build/starling](https://github.com/starling-build/starling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
