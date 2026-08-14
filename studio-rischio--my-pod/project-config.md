---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A native macOS SwiftUI app that syncs music to click-wheel iPods by statically linking a vendored,
modified fork of **libgpod** (LGPL 2.1+). App code is MIT. See [README.md](README.md) for the
user-facing description.

## Build

```bash
./build.sh              # build C deps if needed, build app, launch it
./build.sh build        # same, without launching
./build.sh clean        # clean app build output + libgpod artifacts

CONFIG=Release ./build.sh build      # Debug is the default
```

The first run compiles libgpod (a few minutes). Subsequent runs skip it — `Scripts/build-libgpod.sh`
short-circuits when `Vendor/libgpod/src/.libs/libgpod.a` exists. Force a libgpod rebuild with
`./Scripts/build-libgpod.sh --force`. Xcode builds work too; a "Build libgpod" run-script phase
covers the same ground.

Prerequisites: `brew install glib pkg-config`, plus `brew install autoconf automake libtool gtk-doc
intltool` only when `Vendor/libgpod/configure` still needs generating.

### Cutting a release

```bash
CONFIG=Release ./build.sh build
./Scripts/bundle-app.sh                    # make the .app self-contained
ditto -c -k --sequesterRsrc --keepParent \
  "build/Build/Products/Release/My Pod.app" "My-Pod-<version>-arm64.zip"
```

`bundle-app.sh` exists because **only libgpod is static** — the app still links glib, gobject,
gmodule, intl, libplist and gdk-pixbuf from Homebrew by absolute path, so an unbundled `.app` dies
with "Library not loaded" anywhere Homebrew isn't installed at that exact prefix. The script walks
the dependency graph (the transitive set is 10 dylibs, not 6), rewrites load commands to `@rpath`,
**deletes the Homebrew `LC_RPATH` entries** so a machine that does have Homebrew can't shadow the
bundled copies with an incompatible version, and re-signs — rewriting load commands invalidates the
signature. Use `ditto`, not `zip`; `zip` corrupts the signature.

`Config/MyPod.xcconfig` strips the Release build (`DEPLOYMENT_POSTPROCESSING` + `STRIP_INSTALLED_PRODUCT`
+ `STRIP_STYLE = debugging`, all `[config=Release]`) and turns off
`CODE_SIGN_INJECT_BASE_ENTITLEMENTS`. Without the first three, the executable keeps the linker's
debug map — N_OSO stabs naming every `.o` by absolute path, embedding the builder's home directory
~58 times in a binary that otherwise holds nothing personal. `strings` won't show them; they're in
the symbol table. Without the fourth, Xcode injects `com.apple.security.get-task-allow`, which lets
any process attach a debugger to the shipped app. Verify before uploading — both should print 0:

```bash
APP="build/Build/Products/Release/My Pod.app"
nm -ap "$APP/Contents/MacOS/My Pod" | grep -c OSO
codesign -d --entitlements - "$APP" 2>/dev/null | grep -c get-task-allow
```

Ship **only** the `.app`. The `.dSYM` built beside it still carries the full `DW_AT_comp_dir` build
paths by design — that is what it's for — so it must never go into a release.

Bump `MARKETING_VERSION` in the project and the download button + version line in `docs/index.html`,
which hardcode the asset URL (`releases/download/v<version>/My-Pod-<version>-arm64.zip`). The page
is served by GitHub Pages from `main` → `/docs`, so pushing publishes it.

Everything the page loads must live **inside** `docs/` — Pages serving from `/docs` treats that
folder as the site root and cannot reach `../icon/`. Hence `docs/app-icon.png` and `docs/icon.svg`
are copies; `icon/` remains the design source (the `.afdesign` and its component SVGs).

Releases are arm64-only and ad-hoc signed (there is no `DEVELOPMENT_TEAM`), so users must clear the
quarantine flag. Notarizing would require a Developer ID and turning on hardened runtime, which is
currently off. Because libgpod is statically linked, **any binary release must be accompanied by the
corresponding source** — see the licensing section.

**There is no test target and no tests.** Verification is by building and running against a real
device. The highest-value check before publishing changes is a fresh-clone build from tracked files
only, which catches things `.gitignore` accidentally excludes:

```bash
DEST=$(mktemp -d)/MyPod && mkdir -p "$DEST"
git archive HEAD | tar -x -C "$DEST" && cd "$DEST" && ./build.sh build
```

## Architecture

Four layers, bottom-up:

1. **`Vendor/libgpod/`** — upstream C library that reads/writes the iTunesDB format. Statically
   linked.
2. **`My Pod/IPodKit/ipod-api.{c,h}`** — the only place GLib types appear. Exposes an opaque
   `IPodDB*` plus plain-C structs so nothing above it sees `GList`/`GHashTable`/`GError`. Reached
   from Swift through `My_Pod-Bridging-Header.h`.
3. **`My Pod/Models/IPodDevice.swift`** — a Swift `actor` wrapping `IPodDB*`, one method per C call,
   converting `IPodResult` into thrown `IPodError`s and freeing every returned C string.
4. **Services + Views** — ordinary Swift/SwiftUI. `IPodController` (@MainActor @Observable) owns the
   device lifecycle; `ContentView` wires the four stores together and passes state down.

Data flow for a sync: `VolumeWatcher` (mount notifications) → `IPodController.load` → `IPodDevice`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [studio-rischio/My-Pod](https://github.com/studio-rischio/My-Pod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
