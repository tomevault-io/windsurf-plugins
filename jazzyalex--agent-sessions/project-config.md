---
trigger: always_on
description: - All marketing / promo / growth work coordinates through `Marketing/STATUS.md` — read it first, update it last. It's the source of truth and indexes the detail files (goal + star log, angles, Product Hunt kit, HN draft). Draft only; the owner posts to their own accounts. (`Marketing/` is gitignored and local to this machine.)
---

# Agents Guidelines

## Marketing / Promo Coordination
- All marketing / promo / growth work coordinates through `Marketing/STATUS.md` — read it first, update it last. It's the source of truth and indexes the detail files (goal + star log, angles, Product Hunt kit, HN draft). Draft only; the owner posts to their own accounts. (`Marketing/` is gitignored and local to this machine.)

## Build & Review Discipline
- Do not ask the user to “confirm” or “if it looks good” until the code compiles locally with zero build errors.
- After making changes that affect Swift sources or Xcode integration, validate by building the active scheme.
- If the project cannot be built in your environment, clearly state what prevented the build, and provide the exact file and line references you validated.

## Swift/macOS QA
- If test automation or QA scripts force macOS Appearance to Dark Mode, always restore macOS Appearance back to `System` at the end of the run.
- In Codex Desktop, Swift/Xcode build and test commands commonly need access to Xcode cache directories that are outside the workspace sandbox. For `xcodebuild`, SwiftPM, or XCTest runs, request approved Xcode access up front when the command is expected to touch DerivedData, ModuleCache, SourcePackages, simulator caches, or other Xcode-managed cache paths. If a first run fails only because sandboxing blocked one of those paths, rerun the exact same command with approved Xcode access and report it as a sandbox access retry, not as a code or test failure.
- Prefer narrow approved command prefixes for trusted repo-local Xcode workflows, such as the canonical AgentSessions build/test commands and `./scripts/xcode_test_stable.sh`. Do not use broad auto-approval rules for arbitrary Xcode-looking commands; if a command falls outside the trusted prefixes, request explicit approved Xcode access for that command.
- Relaunching the app to test a change is trivial — just `killall AgentSessions 2>/dev/null; open .deriveddata-manual/Build/Products/Debug/AgentSessions.app` (the output of the manual `xcodebuild … -derivedDataPath .deriveddata-manual build`). Do NOT invoke a "run" skill or hunt for a launch skill — there isn't one and none is needed. Only reach for UI automation (computer-use/screenshots) if explicitly asked to drive or screenshot the app.

## Instructions for Codex CLI



### Format
```
I'll make the following changes:
- File X: Add/modify Y because Z
- File A: Remove B because C

[Immediately proceed with code changes - user has ESC window during explanation]

- Edited file.swift...
```

### Flow Pattern
**Correct:** Explain what will be done → Code → Results

### Examples of What NOT to Do
❌ Don't: Start with "• Edited file.swift..." before explaining
❌ Don't: Ask "Should I proceed?" or wait for confirmation
❌ Don't: Begin analyzing/thinking without stating the plan upfront

### Examples of What TO Do
✅ Do: "I'll tighten probe detection by requiring Probe WD for /status sessions and limiting marker matching. This reduces false positives." [then immediately start coding]
✅ Do: State the approach clearly, then flow directly into implementation
✅ Do: Give user the ESC window by printing plan first, but maintain momentum

### Special Mode
When user says "plan mode++" - ONLY provide the plan and stop. Wait for explicit approval before coding.

This applies to ALL coding requests. The explanation is for transparency and ESC opportunity, not for breaking flow.

### Significant change gating (must build before presenting)
Treat a change as “significant” and always run a build locally before presenting results when any of the following are true:
- Added, moved, or renamed any Swift file (app or tests).
- Modified more than ~40 lines of Swift across the app, or touched 2+ top‑level areas (e.g., Views + Services, Model + Views).
- Introduced or changed concurrency boundaries (actors, Task, async/await), or cross‑module interactions.
- Altered window/layout/toolbar structure or target membership (PBXBuildFile/target Sources).
- Changed build settings, target configuration, Info.plist, or added resources.

It is acceptable to present without building for clearly minor edits, for example:
- One‑line fixes that do not affect types/signatures, string/label copy changes, comment/doc updates, or pure Markdown/JSON assets.
- In case of doubt, prefer to build.

Suggested build steps
- Xcode: Product → Build (active scheme).
- CLI: `xcodebuild -project AgentSessions.xcodeproj -scheme AgentSessions -configuration Debug build` (or use your configured build task).

### Stable XCTest Invocation (avoid intermittent macOS code-sign flakes)
- Prefer the stable test wrapper: `./scripts/xcode_test_stable.sh`.
- Equivalent direct command:
  - `xcodebuild -project AgentSessions.xcodeproj -scheme AgentSessions -configuration Debug -destination 'platform=macOS,arch=arm64' -derivedDataPath "$PWD/.deriveddata-tests" -parallel-testing-enabled NO clean test`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jazzyalex/agent-sessions](https://github.com/jazzyalex/agent-sessions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
