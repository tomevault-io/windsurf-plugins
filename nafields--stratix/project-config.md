---
trigger: always_on
description: This file is the primary working contract for any agent operating in this repository. Read it before touching any file. Subdirectory `AGENTS.override.md` files extend and narrow these rules for their specific scope.
---

# AGENTS.md — Stratix

This file is the primary working contract for any agent operating in this repository. Read it before touching any file. Subdirectory `AGENTS.override.md` files extend and narrow these rules for their specific scope.

**Modernization contract reference:** For any work governed by the structural modernization program, read `Docs/Stratix_Modernization_Contracts.md` together with `Docs/Stratix_Modernization_Plan.md`, `Docs/Stratix_Monolith_Breakdown.md`, and `Docs/Stratix_File_Matrix.md`. The contracts document is the canonical AGENTS reference for the Floor Contract and Execution Contract.

**Skill routing reference:** For any coding, review, build, test, performance, rendering, persistence, or platform API task in this repo, use `[$ios-skills:ios-skills-router](/Users/nicholas/.ios-skills/skills/_router/SKILL.md)` first and then load the repo-relevant skills listed in `Docs/Stratix_Skill_Policy.md` before coding.

---

## Project Overview

**Stratix** is a tvOS 26 SwiftUI application that provides an Xbox cloud gaming client (xCloud / Game Pass) for Apple TV. It is the single app target in this monorepo. The repo also contains seven Swift Package Manager packages that the app target consumes.

- **App target:** `Apps/Stratix/` (product name: Stratix, brand: STRATIX)
- **Packages:** `Packages/` — DiagnosticsKit, StratixCore, StratixModels, InputBridge, StreamingCore, VideoRenderingKit, XCloudAPI
- **Third-party:** `ThirdParty/WebRTC/WebRTC.xcframework` — pre-built binary, tvOS arm64 + simulator
- **Tools:** `Tools/webrtc-build/` — build scripts and tvOS-specific patches for the WebRTC source

The app streams cloud games over WebRTC. The UI layer is SwiftUI throughout. The rendering layer uses Metal. Audio is managed via AVAudioSession with tvOS-specific patches applied to the WebRTC framework.

---

## Build Posture

- **Swift version:** 6.2
- **Strict concurrency checking:** complete (enforced for all targets)
- **Deployment target:** tvOS 26.0
- **Xcode workspace:** `Stratix.xcworkspace`

These are hard constraints. Do not lower the deployment target. Do not weaken the concurrency checking level. Do not add `nonisolated(unsafe)` or `@unchecked Sendable` without a documented reason in the file.

**Available build schemes (in Stratix.xcworkspace/xcshareddata/xcschemes/):**

| Scheme | Purpose |
|--------|---------|
| `Stratix-Debug` | Default development build |
| `Stratix-Profile` | Instruments profiling |
| `Stratix-MetalProfile` | GPU profiling via Metal |
| `Stratix-Perf` | Performance test runs |
| `Stratix-ShellUI` | UI test harness mode |
| `Stratix-ReleaseRun` | Release-configuration local run |
| `Stratix-Packages` | Runs all SPM package tests |
| `Stratix-Validation` | Full validation suite |

**WebRTC compile flag:** `OTHER_SWIFT_FLAGS = -DWEBRTC_AVAILABLE`. All WebRTC-dependent code is wrapped in `#if WEBRTC_AVAILABLE`. Do not remove these guards.

---

## Skill Policy

Agents working in this repo should use repo-matched skills deliberately, not opportunistically after coding has already started.

### Default skill stack

Load these by default for most source changes:

- `[$ios-skills:ios-skills-router](/Users/nicholas/.ios-skills/skills/_router/SKILL.md)`
- `[$ios-skills:swiftui-pro](/Users/nicholas/.ios-skills/skills/twostraws--swiftui-pro/SKILL.md)` for app-target UI and SwiftUI composition
- `[$ios-skills:swift-concurrency-pro](/Users/nicholas/.ios-skills/skills/twostraws--swift-concurrency-pro/SKILL.md)` for all async, actor, and boundary-isolation work
- `[$ios-skills:swift-coding-guideline](/Users/nicholas/.ios-skills/skills/martinlasek--swift-coding-guideline/SKILL.md)` for general Swift/package changes
- `[$ios-skills:swift-accessibility-skill](/Users/nicholas/.ios-skills/skills/pasqualevittoriosi--swift-accessibility/SKILL.md)` whenever touching app UI

### Repo-specific trigger skills

Use these when the file or domain matches:

- `[$ios-skills:swift-testing-expert](/Users/nicholas/.ios-skills/skills/avdlee--swift-testing-expert/SKILL.md)` for new package tests and Swift Testing migrations
- `[$ios-skills:swiftdata](/Users/nicholas/.ios-skills/skills/dpearson2699--swiftdata/SKILL.md)` for `Packages/StratixCore/Sources/StratixCore/Hydration/SwiftDataLibraryRepository.swift` and nearby hydration persistence work
- `[$ios-skills:ios-networking](/Users/nicholas/.ios-skills/skills/dpearson2699--ios-networking/SKILL.md)` and `[$ios-skills:swift-codable](/Users/nicholas/.ios-skills/skills/dpearson2699--swift-codable/SKILL.md)` for `Packages/XCloudAPI/`
- `[$ios-skills:swift-security-expert](/Users/nicholas/.ios-skills/skills/ivan-magda--swift-security-expert/SKILL.md)` for auth and token storage, especially `Packages/XCloudAPI/Sources/XCloudAPI/Auth/TokenStore.swift`
- `[$ios-skills:debugging-instruments](/Users/nicholas/.ios-skills/skills/dpearson2699--debugging-instruments/SKILL.md)` for streaming, rendering, AVFoundation, performance, and runtime investigations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nafields/stratix](https://github.com/nafields/stratix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
