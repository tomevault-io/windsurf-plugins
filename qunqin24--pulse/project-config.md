---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Pulse is a macOS menu-bar app: a floating AI usage monitor. SwiftUI renders the UI; a transparent, non-activating AppKit `NSPanel` anchors it to a side of the screen. It tracks three coding agents — Claude Code, Codex and Antigravity — reading each one's real limits by whatever route that agent offers. There is no backend and no account of its own.

## Commands

```bash
swift run Pulse         # build and run
swift build             # type-check everything, previews included
./Scripts/bundle.sh     # assemble build.noindex/Pulse.app (add --zip for a release)
./Scripts/dmg.sh        # the same, wrapped in build.noindex/Pulse-<version>.dmg
./Scripts/check-localization.sh   # the .strings files agree, and every key the source asks for exists
```

**`xcode-select` must point at Xcode, not CommandLineTools.** The `#Preview`
macro is expanded by a plugin that ships with Xcode; without it every build
fails with `PreviewsMacros plugin not found`. Check with `xcode-select -p`, and
fix a wrong one with:

```bash
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```

(`DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer swift build` works
as a one-off if you can't change the system setting.) Do **not** "fix" this by
stripping the `#Preview` blocks before building: that produces a clean build
while leaving errors inside those blocks completely unreported, which has
already shipped a broken build to the user once.

**A clean `swift build` is not the same check Xcode runs.** Actor-isolation
mistakes can come out as warnings here and as hard errors in Xcode, where they
fail the whole module and every other file then reports "cannot find type X in
scope" — a wall of noise pointing anywhere but the cause. That has happened
once already, from a nonisolated helper reaching for a constant on a SwiftUI
`View` (every `View` is `@MainActor`). Before saying a change builds:

```bash
swift build -Xswiftc -swift-version -Xswiftc 6
```

and treat any remaining warning as a failure.

The package requires macOS 14+ and Swift tools 6.0 (see [Package.swift](Package.swift)). It can also be opened directly in Xcode via `Package.swift`. There is no test target and no linter configured.

## Shipping

There is no Xcode project, on purpose — Pulse is a plain package, and
[Scripts/bundle.sh](Scripts/bundle.sh) is what turns its bare executable into
something macOS treats as an app. That is not tidiness: **without a bundle
there is no `Bundle.main`**, which means no version number to compare against
(so no update check), no `SMAppService` login item (hence the launch-agent
fallback in [LoginItem.swift](Sources/Pulse/LoginItem.swift)), and nothing to
hand anyone but a build folder.

- **Pushing a tag is the whole release.**

  ```bash
  echo 1.0.1 > VERSION && git commit -am "Pulse 1.0.1"
  git tag v1.0.1 && git push && git push origin v1.0.1
  ```

  [`.github/workflows/release.yml`](.github/workflows/release.yml) builds,
  packages and publishes, then signs the archive and commits the Sparkle feed;
  [`ci.yml`](.github/workflows/ci.yml) runs the same build on every push. The
  release needs one secret, `SPARKLE_PRIVATE_KEY`, and fails loudly without it
  rather than publishing a version no installed copy would ever be offered. Both pin **`macos-26`** rather than `macos-latest`,
  because `PanelSurface`'s Liquid Glass needs the macOS 26 SDK to compile at
  all even though it is behind an `#available` check — an older image cannot
  build the package, so both workflows check `xcrun --show-sdk-version` first
  and fail with that sentence instead of a hundred lines of "cannot find
  glassEffect". Warnings fail the build, for the reason at the top of this
  file.   **The tag is checked against `VERSION`** and a mismatch fails the run: ship
  them out of step and every installed copy is told, forever, that an update is
  waiting. A tag with a suffix (`v1.1.0-beta.1`) is published as a pre-release,
  which GitHub's "latest release" excludes — and so, therefore, does the update
  check.
- The version lives in [VERSION](VERSION) and nowhere else. Tag the release
  `v$(cat VERSION)`: the update check reads GitHub's latest release tag and
  compares it against `CFBundleShortVersionString`, so the two have to agree.
- The package's resource bundle must land in `Contents/Resources` —
  `Bundle.module` looks there through `Bundle.main.resourceURL`. Leave it out
  and the app runs, silently in English, with no provider marks.
- `LSUIElement` must be **true**. The code also sets `.accessory`, but that runs
  after the Dock has already been told what to show, so without the key a Dock
  icon appears for a moment at every launch.
- Built for both architectures (`--arch arm64 --arch x86_64`), so one download
  runs everywhere, and zipped with `ditto`, not `zip` — a plain zip flattens the
  symlinks in a bundle and the unzipped copy won't launch.
- **The output directory is named `build.noindex/` for a reason.** Spotlight
  indexes an `.app` wherever it finds one, so a build sitting in the project

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qunqin24/Pulse](https://github.com/qunqin24/Pulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
