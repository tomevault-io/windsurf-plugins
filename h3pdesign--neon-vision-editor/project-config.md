---
trigger: always_on
description: This file is the single source of truth for how Codex must work in this repository.
---

# Codex Project Instructions - Neon Vision Editor

This file is the single source of truth for how Codex must work in this repository.
Primary goal: ship small, safe changes that preserve existing behavior and architecture.

Neon Vision Editor is a native SwiftUI/AppKit editor focused on speed, stability, and minimalism.
Avoid "helpful refactors." Fix or implement exactly what is requested.

---

## 0) Global Non-Negotiables (Always)

### A. Preserve working code
- Do NOT touch working code unless strictly required.
- No opportunistic refactors.
- No renaming/moving for aesthetics.

### B. No deprecated APIs
- Do not introduce deprecated Apple APIs.
- Modern Swift patterns only if required for the task.

### C. Security & Privacy
- No telemetry.
- No sensitive logging (documents, prompts, tokens).
- Network calls only when explicitly user-triggered.
- API tokens must remain in Keychain.
- No weakening sandbox or file security.

### D. Small, reviewable diffs
- Minimal patches only.
- If change is large, split into phases.
- If diff exceeds reasonable review scope, stop and split.

### E. Multi-window correctness
- No accidental shared state across windows/scenes.
- Window state must remain isolated unless explicitly designed otherwise.

### F. Main thread discipline
- UI mutations must be on main thread / `@MainActor`.
- No blocking IO, parsing, or network on main thread.

---

## 1) Hard Cross-Platform Rule (Blocking)

Neon Vision Editor ships on:
- macOS
- iOS
- iPadOS

ANY change targeting one platform MUST prove it does not break the others.

### Mandatory compile safety
- No AppKit types in shared code without `#if os(macOS)` guards.
- No UIKit-only APIs leaking into macOS builds.
- Shared models must remain platform-agnostic.

### Mandatory verification requirement (STRICT)

A patch is NOT acceptable without explicit iOS and iPadOS verification steps.

Your response MUST include either:

A) Exact build commands executed for:
- macOS
- iOS simulator
- iPad simulator

OR

B) A detailed manual verification checklist for:
- macOS
- iOS
- iPadOS

If this section is missing, the answer is incomplete.

### Standard build verification command

Prefer the repo matrix script when Xcode and simulator services are available:

```bash
scripts/ci/build_platform_matrix.sh
```

It runs macOS, iOS Simulator, and iPad Simulator builds sequentially with `CODE_SIGNING_ALLOWED=NO`. By default it writes to `.DerivedDataMatrix` and removes it on exit. Use `--keep-derived-data` only when actively debugging build artifacts, then remove the derived data directory before finishing.

Equivalent individual commands, when the matrix script is not suitable:

```bash
xcodebuild -project "Neon Vision Editor.xcodeproj" -scheme "Neon Vision Editor" -configuration Debug -destination "generic/platform=macOS" -derivedDataPath .DerivedData-macOS CODE_SIGNING_ALLOWED=NO build
xcodebuild -project "Neon Vision Editor.xcodeproj" -scheme "Neon Vision Editor" -configuration Debug -sdk iphonesimulator -destination "generic/platform=iOS Simulator" -derivedDataPath .DerivedData-iOS CODE_SIGNING_ALLOWED=NO build
xcodebuild -project "Neon Vision Editor.xcodeproj" -scheme "Neon Vision Editor" -configuration Debug -sdk iphonesimulator -destination "generic/platform=iOS Simulator" -derivedDataPath .DerivedData-iPad TARGETED_DEVICE_FAMILY=2 CODE_SIGNING_ALLOWED=NO build
```

Remove any `.DerivedData*` folders created during verification before returning.

---

## 2) Accessibility Rule (Non-Optional)

Every UI-affecting change must consider:

- VoiceOver (macOS + iOS/iPadOS)
- Keyboard navigation (macOS + iPad with keyboard)
- Focus management

You must explicitly state:

- What accessibility elements were affected
- How labels/traits remain correct
- That focus order is preserved
- That no UI state traps accessibility focus

If accessibility validation is missing, the patch is incomplete.

---

## 3) Mode Selection (Declare One)

At the top of every implementation response, state:

- `MODE: BUGFIX/DEBUG`
or
- `MODE: NEW FEATURE`

If unclear, default to `BUGFIX/DEBUG`.

---

# MODE: BUGFIX / DEBUG

## Scope
Crashes, regressions, incorrect behavior, UI glitches, build failures, performance issues.

## Rules
- Do not change expected behavior unless clearly wrong.
- Fix smallest surface possible.
- Prefer guards and state corrections over redesign.
- Debug logs must be `#if DEBUG` gated.

## Required Output Structure

1. Repro steps
2. Root cause hypothesis
3. Minimal patch plan
4. Patch
5. Verification
   - macOS
   - iOS
   - iPadOS
   - Accessibility checks (if UI touched)
6. Risk assessment

If any section is missing, the response is invalid.

---

# MODE: NEW FEATURE

## Scope
New UI, new settings, new editor capability, new integration.

## Rules
- Must fit lightweight editor philosophy.
- No IDE bloat.
- Integrate into existing infrastructure.
- No hidden default behavior changes.

## Platform Requirements
- Must define macOS interaction model.
- Must define iOS touch interaction.
- Must define iPad keyboard/multitasking behavior.
- If platform-limited, explicitly guard and document.

## Required Output Structure

1. User problem
2. Proposed solution
3. Why this fits scope
4. Alternatives considered

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [h3pdesign/Neon-Vision-Editor](https://github.com/h3pdesign/Neon-Vision-Editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
