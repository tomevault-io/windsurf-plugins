---
trigger: always_on
description: Guidance for agents working in this repository. `CLAUDE.md` is a symlink to this file.
---

# AGENTS.md

Guidance for agents working in this repository. `CLAUDE.md` is a symlink to this file.

This file is the always-on contract. Put ticket history, platform essays, and research notes elsewhere — update this file only when a durable agent rule changes.

## Working together

- **Follow through:** infer routine details from the current request and verified context, state material assumptions, and finish authorized work. Ask only when missing input changes scope, correctness, or authorization; continue independent work while waiting. Treat mid-task corrections as updates to the active task unless the user cancels it.
- **Delegate:** for substantial work, the lead agent clarifies scope, decomposes tasks, dispatches, and accepts results. Use available subagents for extensive reading, implementation, batch edits, and test execution; parallelize independent work. Give each worker a bounded task, file ownership, constraints, and acceptance evidence. Workers share the workspace: preserve others' edits. Keep trivial tasks local unless the user requests delegation; if delegation is unavailable, report the limitation and honor any explicit role restriction.
- **Verify:** accept worker results against the current diff and relevant test or runtime evidence. Run checks proportional to the change and required gates; repeat only after relevant edits, failures, or unresolved concerns. Documentation-only changes need static checks, not Gradle/Xcode. UI/performance claims still need the applicable visual and physical-device evidence; record missing evidence as pending.
- **Communicate:** use concise, plain language in user and agent messages. Lead with the outcome, explain relevant changes and checks, and state remaining limitations without repeated status or invented labels.
- **Scope:** preserve existing work and authorization boundaries. Tool permissions are not permission to publish, send messages, merge, or perform destructive cleanup; obtain user authorization when it is not already present. Historical mission files are evidence, not current instructions, unless the user explicitly resumes that mission; reconcile their rules with the current request and this file first.

## Product

EasyWatermark (`me.rosuh.easywatermark`) tiles text or image watermarks over photos so they cannot be reused. Fully offline; no tracking, stats, or crash SDKs. One Kotlin Multiplatform / Compose Multiplatform codebase ships Android, Desktop (JVM), and iOS.

Privacy that shapes code: Android needs no runtime permission on API 29+ (pre-29 storage). iOS pick needs no library read; save is add-only; optional Library Read is a photo-layer latch under a matching overlay (ADR-0029 + ADR-0033). Session and export stay path-first (ADR-0021). Export strips all EXIF (ADR-0009). Android ships via GitHub Releases, Google Play (paid, same code), F-Droid, and Coolapk.

## Modules

| Module | Role |
|---|---|
| `:shared` | Cross-platform domain, Room, session, render, Compose UI (`android` + `desktop` + iOS) |
| `:app` | Android shell: Activity, ports, MediaStore/decode/save, Koin |
| `:desktopApp` | Compose Desktop window + `--headless` CLI |
| `iosApp` | SwiftUI shell; `Shared.framework`; PHPicker / Photos / share |
| `:cmonet` | Android wallpaper Material You only, behind `DynamicColorCapability` |

`commonMain` is Kotlin + Compose with no Android types. Platform source sets own DataStore/Room builders, decode/encode, and system I/O.

## Read when

| When | Open |
|---|---|
| Domain words, invariants, retired terms | `docs/CONTEXT.md` |
| A design fork or “why is it this way” | `docs/adr/` |
| Issue / handoff / what not to recreate | `docs/agents/issue-tracker.md` |
| Agent guidance maintenance / model migration scope | `docs/agents/workflow.md` |
| GitHub label names | `docs/agents/triage-labels.md` |
| Android / KMP API (not training data) | `android docs search '<query>'` then `android docs fetch` |

Do not start sessions from `task_plan.md`, `findings.md`, `progress.md`, or `docs/superpowers/research/`.

## Commands

```bash
./gradlew :app:assembleDebug
./gradlew :app:testDebugUnitTest
./gradlew :shared:desktopTest
./gradlew :shared:iosSimulatorArm64Test          # macOS only
./gradlew :app:connectedDebugAndroidTest
./gradlew :desktopApp:run
./gradlew :desktopApp:run --args='--headless'
./gradlew :desktopApp:run -PewmAutoOpen=<abs image>
```

Debug `applicationId` is `me.rosuh.easywatermark.debug` (installs beside production). SDK: `Apps.compileSdk` 37, `targetSdk` 36, `minSdk` 23, JVM 17. No Spotless/ktlint — match existing style. PR CI: Ubuntu `assembleDebug` + `desktopTest` + non-strict `testDebugUnitTest`; macOS iOS job. Docs/assets-only PRs still start `PR Checks` so the two required job names report success; Gradle/Xcode run only when a product path changes. `lintDebug` is fail-open. Do not add `WATERMARK_GOLDEN_STRICT=true` to PR CI (ADR-0010). Unsigned Desktop packaging is not a PR required check (ADR-0031).

## Rules

Pair every “don’t” with the replacement.

- **UI:** new product UI in `shared/commonMain/ui/`. Native UI only for app/window entry, pickers, share/save/permissions, capability glue, and renderer surfaces. Do not reintroduce `ViewInfo` or an `AndroidView` renderer.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rosuH/EasyWatermark](https://github.com/rosuH/EasyWatermark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
