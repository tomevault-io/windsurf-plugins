---
trigger: always_on
description: - Risper is a local-first Hebrew dictation utility for macOS.
---

# AGENTS.md

## Repository Context

- Risper is a local-first Hebrew dictation utility for macOS.
- Runtime audio processing must stay local for the MVP. Do not introduce cloud ASR, telemetry, or transcript upload paths unless the user explicitly changes the product direction.
- Treat `specs.md` as the product and architecture source of truth.
- The app is a SwiftPM-first AppKit macOS app. Use Swift, AppKit, AVFoundation, Carbon/CoreGraphics, and local `whisper.cpp` integration before adding new frameworks.

## Agent Workflow

- Before implementation, read the relevant parts of `specs.md`, `Package.swift`, scripts, and source files touched by the task.
- For macOS app development, always use the `Build macOS Apps` plugin and its relevant task-specific skills before planning or editing code.
- Keep changes small, coherent, and reversible. Separate broad structural refactors from feature or bug-fix changes unless the refactor is required to complete the task safely.
- When the user asks to fix several code-review findings, launch sub-agents to handle independent fixes in parallel where the work can be cleanly separated. Keep tightly coupled or blocking fixes local, and integrate and verify all results before finishing.
- Do not overwrite or revert unrelated user changes. If the worktree is dirty, work with the existing changes.

## Build And Verification

- Run `swift build` after Swift source changes.
- Run `script/build_and_run.sh --verify` when app-bundle behavior, launch behavior, permissions, menus, hotkeys, or lifecycle code changes.
- Use `script/asr_harness.sh` or the local ASR server workflow for transcription-path validation when ASR behavior changes.
- For microphone, hotkey, Accessibility, clipboard, or cross-app paste behavior, document any manual QA that could not be completed from the agent environment.
- Do not add production dependencies, package managers, generated projects, or new build systems without explicit user approval.

## Regression And Debugging Guardrails

- When the user reports that the last commit worked, compare current runtime source files against `HEAD` before changing behavior. Treat the last-known-good runtime path as the baseline.
- For regressions that appear after packaging, signing, bundling, or installer work, first inspect bundle identity, signing, `Info.plist`, installed app path, and TCC permission state before changing Swift runtime logic.
- Keep packaging fixes separate from app-behavior fixes. Do not replace a working event-monitoring, permission, recording, or paste mechanism unless direct evidence shows that mechanism is the cause.
- For microphone, Accessibility, hotkey, overlay, clipboard, or paste changes, verify the real workflow end to end: focus a text field, hold `fn`, confirm the recording indicator appears, release, wait for transcription, and confirm text is inserted at the original cursor.
- Do not treat partial logs as success. Logs that show monitor startup, recording, transcription, or paste events must be paired with manual workflow verification when the behavior depends on macOS focus, permissions, or cross-app input.
- Read `docs/debugging-macos-permissions.md` before diagnosing or changing macOS permissions, app signing, TCC, global hotkey, Accessibility, microphone, clipboard, or cross-app paste behavior.

## Engineering Standards

- Optimize first for clarity, then simplicity, then concision. Code should be easy for the next engineer or agent to read, test, and change.
- Keep responsibilities cohesive. Avoid god objects, god files, and utility dumping grounds. When behavior naturally splits into separate domains, extract along stable domain boundaries rather than by incidental implementation detail.
- Maintain a single source of truth for configuration, paths, hotkeys, permission state, model/server settings, transcript state, and temp-file policy. If data is cached, derived, or duplicated for performance, make the owner and lifetime explicit.
- Avoid speculative generality. Build the behavior required by `specs.md`; defer abstractions until they remove real duplication or clarify a real boundary.
- Prefer proven platform APIs and existing project scripts over bespoke DIY implementations. If custom machinery is necessary, keep it narrow and explain the reason in code or docs.
- When implementing or fixing behavior, identify the underlying invariant, lifecycle, or failure mode before editing. Solve the class of problem across the relevant local surface, not only the visible symptom. For resources, permissions, temporary state, user data, process/global state, and async flows, define acquisition, mutation, cleanup, cancellation, failure, and interruption behavior explicitly where relevant.
- Keep code secure by default: prefer least-privilege permissions, narrow local-only exceptions, validated inputs, safe process/network boundaries, and privacy-preserving failure behavior.
- Reduce toil deliberately. Repeated setup, verification, diagnostics, and recovery steps should become reliable scripts or app diagnostics when they have clear ongoing value.
- Keep documentation short, current, and linked to canonical sources. Update docs in the same change as behavior changes when the docs would otherwise become stale.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shafnir/Risper](https://github.com/shafnir/Risper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
