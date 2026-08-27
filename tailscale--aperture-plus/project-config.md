---
trigger: always_on
description: Working notes for AI coding agents (and humans) on this repo. Read alongside
---

# CLAUDE.md

Working notes for AI coding agents (and humans) on this repo. Read alongside
`README.md`, which has the full build/setup instructions. This file captures the
non-obvious things that are easy to get wrong.

## What this is

A SwiftUI browser (WebKit) that routes traffic through an embedded userspace
Tailscale node (`TailscaleKit` / `libtailscale`). The shipping iPhone/iPad target
and scheme are **`Aperture`**. A native macOS target/scheme, **`ApertureMac`**,
shares the browser/workspace implementation and is being polished incrementally;
see `TODO.mac.md`. Both products use bundle ID `io.tailscale.Aperture`.

## Hard constraints (will break the build if ignored)

- **iOS 26.0 and macOS 26.0 only.** The iOS target retains
  `IPHONEOS_DEPLOYMENT_TARGET = 26.0`, `SDKROOT = iphoneos`; the native Mac target
  uses `MACOSX_DEPLOYMENT_TARGET = 26.0`, `SDKROOT = macosx`. Keep platform-only
  APIs behind target membership, `canImport`, or availability guards.
- **Future Virtualization use requires native macOS.** `ApertureMac` carries
  `com.apple.security.virtualization = true` now, but contains no VM code yet.
  Do not convert it to Catalyst or remove the entitlement. The intended guest is
  pure Linux with app-supplied userspace networking and no shared filesystem.
- **Swift 6 strict concurrency.** `SWIFT_STRICT_CONCURRENCY = complete` and
  `SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor`. The whole module is implicitly
  `@MainActor`-isolated unless you opt out. New code must be concurrency-clean
  (Sendable annotations, explicit `@GlobalActor`/`nonisolated` where needed).
- **The app embeds `TailscaleKit.xcframework`**, which is **not in git**. It must
  exist at `ThirdParty/libtailscale/swift/build/Build/Products/Release-iphonefat/TailscaleKit.xcframework`
  before the project will build. If a build fails with a missing-framework /
  file-not-found error on `TailscaleKit.xcframework`, run
  `cd ThirdParty/libtailscale/swift && make ios-fat` (needs Go 1.26.5). The
  `FRAMEWORK_SEARCH_PATHS` in the project also references `Release-iphoneos`; the
  xcframework reference itself points at `Release-iphonefat` — both are produced by
  `make ios-fat` / `make ios` / `make ios-sim`.
- **`ThirdParty/libtailscale` is a git submodule**, and it contains the nested
  `tailscale-patched` submodule. They are not vendored copies. After cloning,
  run `git submodule update --init --recursive`. Commit changes deepest-first
  (`tailscale-patched`, then `libtailscale`, then this repo) so every gitlink
  points at a real commit.
- **Run `make subtrac` after every top-level commit**, especially any commit that
  changes a submodule pointer. The submodule remotes are intentionally local
  (`url = .`), so ordinary gitlinks alone are not portable. `make subtrac`
  preserves HEAD and recursively embeds both nested commits in `<branch>.trac`
  (normally `main.trac`). It is content-addressed/idempotent and verifies the
  tracking ref equals `git-subtrac cid HEAD`, so running it after every commit
  is safe and cheap. Do not commit the generated tracking commit onto `main`;
  it lives on `main.trac`.

## Adding source files (do NOT hand-edit project.pbxproj for new files)

`App/`, `MacApp/`, and `TSNet/` are Xcode **synchronized folder groups**
(`PBXFileSystemSynchronizedRootGroup`). New `.swift` files dropped into either
directory are automatically compiled into the `Aperture` target — no
`project.pbxproj` editing required. The `UITests/` directory is the same kind of
synchronized folder group, but for the `ApertureUITests` target.

The `TSNet/` group is different: it has a `membershipExceptions` list in
`project.pbxproj` naming the files that ARE compiled into the `Aperture` target,
and in this project's configuration that list **does** gate compilation — a new
`.swift` file dropped into `TSNet/` is NOT picked up until you add its name to the
`membershipExceptions` list (e.g. `TSNet/CrashCapture.swift` had to be added).
(`App/` and `UITests/` have no such list for the iOS app, so files there are
 auto-compiled. `App/` has macOS membership exceptions for iOS-only entry points
 and harnesses; `MacApp/` compiles only into `ApertureMac`.)

Other files (Info.plist, README.md, assets) are normal pbxproj references and do
require project edits if you add/relocate them.

## UI automation & agent tooling

There is a UI test target (`ApertureUITests`; sources in `UITests/`, another
synchronized folder group) plus helpers for running tests, capturing libtailscale
logs, letting a non-vision agent "see" the app, and the optional Xcode MCP server.
All of that — setup steps, the run-destination matrix (simulator vs "My Mac"),
vision-model config, CLI-vs-MCP guidance, and a scripts reference — is documented
in **[`README.ui-automation.md`](README.ui-automation.md)**. Read that when working
on tests, logs, vision, or the MCP bridge.

Headline for always-context: **use the simulator for autonomous work** (build +
`simctl install`/`launch` + `simctl io booted screenshot` + XCUITest + `log stream`
all work with zero permission grants). The "My Mac (Designed for iPad)" target
can't be launched headlessly. The UI tests include a `testHomePageLoadsWhenConnected`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tailscale/aperture-plus](https://github.com/tailscale/aperture-plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
