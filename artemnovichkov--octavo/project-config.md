---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Octavo — a native macOS replacement for calibre (1 GB) whose job is syncing an ebook library to a
Kindle Paperwhite 12 and editing metadata. SwiftPM package, no external dependencies: MTP is written
directly on IOUSBHost, SQLite and zlib come from the SDK. The built `.app` is ~2.9 MB, most of the
growth over the original 1.7 MB being `Octavo.icns`.

Two icons exist side by side, drawn by sibling scripts from the same sheet/dog-ear geometry:
`Resources/Octavo.icns` (`make-icon.swift`), which `make-app.sh`'s release build embeds, and
`Resources/Octavo.icon` (`make-liquid-icon.swift`), a Liquid Glass `.icon` bundle — icon.json plus
a transparent-background glyph PNG in `Assets/`, editable in Icon Composer.app (ships inside
Xcode 26) — which only the Tuist-generated `Octavo.xcworkspace` target consumes, via
`ASSETCATALOG_COMPILER_APPICON_NAME` in `Project.swift`. The plate is not rasterized for the
`.icon`: its gradient is `fill`/`fill-specializations` in icon.json, and the system renders the
rounded-square mask, glass material and specular sheen around the one foreground glyph layer.

It operates **in place on the user's real calibre library** at `~/Calibre Library` (42 books) — same
`metadata.db`, same folder layout, so calibre keeps working alongside it.

**calibre the application is not required.** `CalibreLibraryStore.create(at:)` lays down calibre's
own schema (`CalibreSchema.sql`, copied verbatim from `metadata_sqlite.sql`, `user_version=26`), so
a library Octavo made opens in calibre later with no migration. Which library is open is resolved by
`LibraryLocation`: the path remembered under `LibraryRoot` in the `org.octavo.Octavo` defaults suite
(a suite, not `.standard`, so the CLIs see what the app sees), else `~/Calibre Library` if it exists,
else nothing — and *nothing* is the welcome screen (`AppModel.LibraryState.needsSetup`), which is
deliberately distinct from `.failed`: one offers to create a library, the other reports a broken one.

## Commands

```sh
swift build                              # libraries + CLIs
swift test                               # whole suite, offline: the catalogue tests are skipped
OCTAVO_NETWORK_TESTS=1 swift test        # …including the three tests that hit the live catalogues
swift test --filter convertsEPUB         # one test (regex on the test function name, not a shell glob)
./Scripts/make-app.sh                    # build/Octavo.app (release build + bundle + ad-hoc sign)
OCTAVO_VERSION=1.2 OCTAVO_BUILD=7 ./Scripts/make-app.sh   # what the release workflow does
open build/Octavo.app
swift Scripts/make-icon.swift            # redraws Resources/Octavo.icns; only after changing artwork
swift Scripts/make-liquid-icon.swift     # redraws Resources/Octavo.icon (Liquid Glass); only after changing artwork
open Package.swift                       # opens the package in Xcode; the Octavo scheme runs, but without a bundle
tuist generate                           # builds Octavo.xcworkspace from Project.swift: Octavo runs as a real .app (bundle id, Info.plist, Resources/Octavo.icon) + one scheme per test target; gitignored, rerun after Project.swift/Package.swift changes
```

CLIs, all of which talk to a connected Kindle:

```sh
.build/debug/mtpprobe                    # USB descriptors, MTP session, device contents — read-only
.build/debug/mtpprobe --ls system        # list a folder;  --cat path  dumps a file to stdout
.build/debug/mtpprobe --push f  --rm n   # write/delete a single file in documents/
.build/debug/octavo-sync                 # dry run: what a sync would do, writes nothing
.build/debug/octavo-sync --apply         # real sync (pulls a documents/ backup first unless --no-backup)
.build/debug/octavo-sync --library PATH  # a library other than the resolved one
.build/debug/octavo-convert book.epub    # conversion alone
```

**The MTP interface is claimed exclusively.** While Octavo.app runs, every CLI fails with "The MTP
interface is busy in another process", and vice versa. `pkill -x Octavo` before using the CLIs.

### Releasing

`.github/workflows/release.yml` fires on a `v*` tag: it stamps the tag into the bundle through
`OCTAVO_VERSION`, archives with `ditto -c -k --keepParent` (plain `zip` drops the xattrs an ad-hoc
signature relies on), re-verifies the unpacked copy, and publishes the zip plus a CLI tarball and
`SHA256SUMS.txt`. It is **not idempotent** — a failed run after the tag was pushed needs
`gh release delete <tag> --yes` and the tag deleted on both sides before re-tagging. Both workflows
pin `runs-on: macos-26`; an unpinned `macos-latest` would let the toolchain move between a green CI
run and a release build. The shipped app is ad-hoc signed and not notarized, so users must clear
`com.apple.quarantine` — the README says so, and `Control-click ▸ Open` is not an alternative on
macOS 15+.

## Architecture

```
MTPKit        PTP/MTP over IOUSBHost: transport, container codec, session, operations, hotplug watcher
CalibreLibrary  read/write metadata.db in calibre's schema, book import, metadata.opf

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [artemnovichkov/Octavo](https://github.com/artemnovichkov/Octavo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
