---
trigger: always_on
description: **IMPORTANT: Before starting any implementation or creating a plan, ask clarifying questions using the AskUserQuestion tool (not plain text).** Always use the AskUserQuestion tool in Claude Code when you have questions to ask after a prompt. Use it to understand:
---

# NewsBlur Development Guidelines

## Planning & Clarification
**IMPORTANT: Before starting any implementation or creating a plan, ask clarifying questions using the AskUserQuestion tool (not plain text).** Always use the AskUserQuestion tool in Claude Code when you have questions to ask after a prompt. Use it to understand:
- The specific goals and desired outcomes
- Edge cases and error handling preferences
- UI/UX preferences (if applicable)
- Performance or scalability requirements
- Integration points with existing code
- Testing expectations
- Any constraints or preferences I might have

**Codex: Use the `request_user_input` tool frequently throughout development - not just during planning.**
**Claude Code: Always use the AskUserQuestion tool (not plain text questions) when you need to ask the user something - during planning, implementation, and any other time.**

Actively interview the user at any point (especially during planning). Prefer multiple rounds of short questions.

Asking questions is encouraged and appreciated because it:
- Helps both of us think through problems more clearly
- Surfaces edge cases and requirements that might be missed
- Leads to better solutions through collaborative dialogue
- Catches misunderstandings early before code is written

Ask about:
- Clarifying requirements and desired behavior
- UI/UX preferences and design decisions
- Trade-offs between different approaches
- Edge cases and error handling
- Whether a proposed solution matches expectations
- Anything you're uncertain about

Don't assume - ask. Multiple rounds of questions are better than one large batch. Even mid-implementation, if something feels unclear or you're choosing between options, ask. The interactive back-and-forth is valuable.

## Debugging

For debugging sessions: always take a screenshot first, reproduce the issue, then form a hypothesis before changing code. Do not start editing until the root cause is identified.

## Bug Fixing Workflow
When I report a bug, don't start by trying to fix it. Instead, start by writing a test that reproduces the bug. Then, have subagents try to fix the bug and prove it with a passing test.

## Git Branching
- **Do not automatically create branches**: Work on the current branch unless I explicitly ask you to create or switch to a branch. If you think a branch would help, ask first.

## Platform-Specific Guidelines
- **iOS**: See `clients/ios/CLAUDE.md` for iOS simulator testing and development
  - **Reuse the existing booted simulator**: before any iOS build/install/launch/screenshot flow, run `python3 clients/ios/run_ios.py list` and use the already booted simulator UDID. Build, install, and launch onto that same simulator. Do not create, clone, boot, or switch to a new simulator unless no usable simulator is booted or a clean device state is explicitly needed.
  - **All new iOS files must be written in Swift** (not Objective-C)
- **Android**: Use the adb/emulator workflow below for emulator testing and screenshots
  - **Reuse the existing running emulator**: before any Android build/install/launch/screenshot flow, run `adb devices -l` and use the already attached emulator/device. Install and launch onto that same target. Do not start a new emulator unless no usable emulator/device is attached or a clean device state is explicitly needed.
  - **Commit early and often for Android work**: make frequent commits throughout implementation so there is a clear trail of changes; commit freely rather than batching large edits.
  - **Theme coverage is required for Android UI work**: any Android UI change must be checked against all supported NewsBlur themes, not just the current device theme.
  - Current required Android theme coverage: `light`, `dark`, and `black`.
  - Upcoming theme requirement: account for `Sapia` in any new Android UI work so theme-specific colors are centralized and easy to extend when that theme lands.
  - **Tag every Android version bump commit**: whenever a commit changes `versionName` or `versionCode` in `clients/android/NewsBlur/app/build.gradle.kts`, tag that exact commit as `Android_<versionName>` (e.g. `Android_14.2.2`) and push the tag. This keeps the `Android_*` tag series consistent so each released build can be traced back to its source.

## Android Emulator Testing
- Prefer reusing an already running emulator instead of booting a new one
- Check attached devices with `adb devices -l`
- If no emulator is attached, start one from the repo root with `make android-emulator`
- The current app package is `com.newsblur`
- The launcher activity resolves to `com.newsblur/.activity.InitActivity`
- Do not rely on `emulator -list-avds` being available on `PATH`; in this repo it was not

### Build and Install
- Run Android Gradle commands from `clients/android/NewsBlur`
- Set `JAVA_HOME` explicitly before Gradle commands or the build may pick the wrong JDK and fail with `invalid source release: 21`
- Working command: `env JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home" ./gradlew :app:installDebug`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samuelclay/NewsBlur](https://github.com/samuelclay/NewsBlur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
