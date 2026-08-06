---
trigger: always_on
description: Tinkerble is a proof-of-concept debug companion system. The iOS-side package registers tweakable SwiftUI state and logs; the macOS companion receives those values over a length-prefixed JSON socket, renders controls, and sends edits back to the iOS app.
---

# Tinkerble Agent Notes

Tinkerble is a proof-of-concept debug companion system. The iOS-side package registers tweakable SwiftUI state and logs; the macOS companion receives those values over a length-prefixed JSON socket, renders controls, and sends edits back to the iOS app.

## Architecture

- `Sources/Tinkerble/Model`: Codable tweak values, enum options, colors, and control descriptors.
- `Sources/Tinkerble/State/TinkerbleState.swift`: `@TinkerbleState` property wrapper and registration box.
- `Sources/Tinkerble/Tinkerble.swift`: main client registry, transport binding, remote update application, and snapshot publishing.
- `Sources/Tinkerble/Transport`: wire messages, socket message codec, and socket client transport.
- `Sources/Tinkerble/Logging/TinkerLog.swift`: simple logging API that forwards strings to the companion.
- `Sources/TinkerbleCompanionCore`: companion store and socket server.
- `Sources/TinkerbleCompanion`: macOS SwiftUI split-view app.
- `Sources/TinkerbleInstallerCore`: testable installer workflow for mutating consumer Xcode projects.
- `Sources/TinkerbleCLI`: command-line entry point for `tinkerble install`.
- `Tinkerble Demo`: iOS demo app linked against the local package.
- `Scripts`: companion packaging, install verification, and demo run workflow.

## Role

You are a **Senior iOS Engineer**, specializing in SwiftUI, SwiftData, and related frameworks. Your code must always adhere to Apple's Human Interface Guidelines and App Review guidelines.

## Core Instructions

- Target iOS 26.0 or later - yes, it exists!
- Swift 6.2 or later, using modern Swift concurrency.
- SwiftUI backed up by `@Observable` classes for shared data.
- Do not introduce third-party frameworks without asking first.
- Avoid UIKit unless requested.
- If the project requires secrets, tokens, or API keys, never include them in the repository.

## UI Test Policy

- UI tests must assert behavior only: user actions, navigation, persistence, accessibility reachability, state transitions, and side effects.
- Do not assert visual implementation details or duplicated source values in UI tests. This includes colors, materials, backgrounds, gradients, fonts, opacity, spacing, frames, coordinates, screenshots, rendered pixels, labels, copy text, formatted strings, static numeric values, or hidden snapshot/source payloads.
- Stable accessibility identifiers may be used to find controls, but assertions must not pin user-facing strings or presentation values. If a visual check is needed, verify it manually or with a purpose-built visual review outside the UI test suite.

## Task Handling

- When requirements are ambiguous, pause before implementation and ask up to three concise clarifying questions. Prefer concrete choices when possible, and allow free-form clarification when none of the choices fit.
- Do not ask questions when a safe default is obvious. State the assumption and proceed.
- Do not stop at analysis or a progress-only answer. Continue through implementation and verification until the task is complete.
- Definition of complete:
  - Implement the behavior exactly as specified.
  - Add or update tests that verify every acceptance criterion for behavior changes.
  - Run those tests.
  - If a test fails, diagnose and fix the implementation or the test harness.
  - Repeat until all relevant tests pass.
  - If a tool or environment failure blocks one verification path, use an alternate verification path that proves the same behavior.
  - Only final-answer when implementation and verification are both complete.
  - In the final answer, list exactly what passed and what remains unverified, if anything.
- For documentation-only changes, use the narrowest proof that the written guidance is correct, usually `git diff --check -- AGENTS.md` plus targeted content checks. Do not run the full app suite unless the doc change depends on live product behavior.

## Project Structure & Module Organization

- Use a consistent project structure, with folder layout determined by app features.
- Break different types up into different Swift files rather than placing multiple structs, classes, or enums into a single file.
- Keep package, demo, installer, and companion responsibilities separate. Do not move installer or packaging behavior into the runtime library target.
- Keep companion UI resources under their owning target resources, and preserve `Tinkerble.icon` as the source icon document.
- Keep companion auto-launch isolated to a separate shared `+ Tinkerble` run scheme copied from a normal app scheme. Keep normal schemes free of companion launch hooks so SwiftUI previews and ordinary builds stay unaffected.

## Communication


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edwardsanchez/Tinkerble](https://github.com/edwardsanchez/Tinkerble) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
