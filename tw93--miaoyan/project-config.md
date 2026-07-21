---
trigger: always_on
description: > `CLAUDE.md` is a symlink to this file. Claude Code and Codex (and any other
---

# MiaoYan Agent Guide

> `CLAUDE.md` is a symlink to this file. Claude Code and Codex (and any other
> agent reading `AGENTS.md`) share this single source so the guide stays
> in sync.
>
> Claude-specific assets:
> - 全局规则: `~/.claude/CLAUDE.md`
> - Swift 通用规则: `~/.claude/rules/swift.md` (项目级补充 `.claude/rules/swift.md`)
> - 发版 runbook: `.claude/skills/release` (`/release`)
> - Lint: `.claude/skills/lint`
> - App Store 流程: `.claude/skills/appstore`

## Project

MiaoYan is a lightweight Markdown editor built with Swift. The main app is macOS/AppKit, and the repository also contains an iOS target under `MiaoYanMobile/`.

## Tech Stack

- **Markdown rendering**: `swift-cmark-gfm` parses GitHub Flavored Markdown.
- **Syntax highlight**: `Highlightr`.
- **Math / diagrams**: LaTeX formulas, Mermaid, PlantUML supported via the preview renderer.
- **Slide mode**: based on Reveal.js. `---` separators delimit slides.
- **Note storage**: filesystem-backed with folder nesting, file-system watch, auto-save, and version history.
- **Editor**: live preview, syntax highlight, keyboard shortcuts, Prettier-integrated auto-format.
- **iOS target**: SwiftUI under `MiaoYanMobile/`, sharing core models in `Business/` with the macOS app.

## Repository Map

- `Controllers/` - view controllers and window controllers.
- `Views/` - UI components.
- `Business/` - models and business logic.
- `Helpers/` - utilities and services.
- `Extensions/` - Swift extensions.
- `Resources/` - bundled resources.
- `MiaoYanMobile/` - iOS app target, SwiftUI views, mobile services, and mobile resources.
- `MiaoYan.xcodeproj/` - Xcode project and version settings.
- `Package.swift` - Swift package dependency declarations and supported platforms.
- `scripts/` - local build, App Store, release, and project maintenance scripts.
- `scripts/release-ci/` - release note rendering, appcast, notarization, and package helpers.
- `.github/RELEASE_NOTES.md` - public release note source for GitHub release and appcast body generation.
- `.github/workflows/` - sponsor asset maintenance workflows; release builds are not currently driven by a tracked release workflow.

## Commands

```bash
xcodebuild -project MiaoYan.xcodeproj -scheme MiaoYan -configuration Debug build
xcodebuild clean
xcodebuild test -project MiaoYan.xcodeproj -scheme MiaoYan -destination 'platform=macOS' CODE_SIGNING_ALLOWED=NO
xcodebuild -project MiaoYan.xcodeproj -scheme MiaoYanMobile -configuration Debug -destination 'generic/platform=iOS' CODE_SIGNING_ALLOWED=NO build
swiftlint lint --strict
swift-format lint --recursive .
bash scripts/build.sh
bash scripts/build-appstore.sh
ruby scripts/add_tests_target.rb     # only when re-wiring MiaoYanTests after pbxproj reset
ruby scripts/wire_helper_files.rb    # only when re-wiring orphan Diagnostics/UIDelay/AppEnvironment
```

Use the narrowest relevant command first. Full app builds are the default verification for Swift or project changes.

The Xcode project uses classic pbxproj groups (no filesystem-synchronized
groups). Adding a new source file requires manual registration at 4 sites in
`MiaoYan.xcodeproj/project.pbxproj`: a `PBXBuildFile` entry, a
`PBXFileReference` entry, the owning group's `children` list, and the target's
`PBXSourcesBuildPhase` files list. Mimic an existing sibling entry and use a
fresh unique 24-hex ID for each new object.

## Testing

Unit tests live under `MiaoYanTests/`. Coverage targets pure-logic surfaces
(`ImageLinkParser`, `WikilinkIndex.updateNote`, `String+`, etc.). UI flows are
verified by manual smoke after build, not by XCUITest.

Add a new test:

1. Create `MiaoYanTests/<Subject>Tests.swift` (XCTest, `@MainActor` on the
   test methods if they touch `@MainActor`-isolated types; `setUp()` overrides
   cannot be `@MainActor`, construct isolated objects inside the tests).
2. Register the file manually at the same 4 pbxproj sites as an app source,
   but into the `MiaoYanTests` group and the `MiaoYanTests` target's
   `PBXSourcesBuildPhase`. Mimic the `NoteFrontmatterTests.swift` sibling
   entries. `scripts/add_tests_target.rb` is a no-op once the target exists
   (it only bootstraps the target after a pbxproj reset), and the `xcodeproj`
   gem it needs is not installed on this machine anyway.
3. Run `xcodebuild test ...` locally, then push.

`CODE_SIGNING_ALLOWED=NO` is required on the local test command because
the dev signing identity used for `MiaoYan.app` and the per-developer
identity used for `MiaoYanTests.xctest` end up with different Team IDs,
which makes dyld refuse to load the test bundle into the host app. CI
uses a clean runner where signing is consistent, so `ci.yml` does not
pass this flag.

## CI

`.github/workflows/ci.yml` runs on every PR and push to `main`:

- macOS Debug build (no signing required)
- iOS Debug build for `MiaoYanMobile`
- SwiftLint (`--strict`) + swift-format lint
- Release-notes rendering smoke (`scripts/release-ci/notes_to_html.sh` and
  `render_release_body.sh`) so a broken `.github/RELEASE_NOTES.md` is caught
  before release time, not during it
- On tag pushes (`V*`): version-triplet consistency check
  (`MARKETING_VERSION == CURRENT_PROJECT_VERSION == tag`) to prevent the
  V3.5.1 / #524 incident recurrence


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tw93/MiaoYan](https://github.com/tw93/MiaoYan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
