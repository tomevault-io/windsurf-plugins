---
trigger: always_on
description: - `main` is the current Transcripted product, derived from the earlier Draft codebase.
---

# Transcripted Agent Guide

## Current repo truth

- `main` is the current Transcripted product, derived from the earlier Draft codebase.
- The current app on `main` supports **dictation** and **meetings**.
- Meeting capture includes local mic + system audio, imported-audio transcription, optional local-speaker review, and agent-readable Markdown output.
- The older draft / ghostwriting flow is not active on `main`. `DictationSessionController` keeps compatibility stubs for removed draft-mode entry points.
- `Sources/TranscriptedCore/` is an in-repo library consumed through `Sources/Meeting/`. Keep it as a library boundary.
- `Sources/Speech/` owns the app-owned local STT path. Meetings reuse that path through `Sources/Meeting/MeetingSTTAdapter.swift`.
- `Sources/Reliability/` owns wake / sleep recovery for hotkeys and active capture flows.
- `build.sh` builds the app target. The root `Package.swift` exists for `TranscriptedCore` package tests and smoke coverage, not as the main app build.

## Response voice

- Write like a real person texting a friend, not like a presentation.
- Keep things simple, direct, and useful.
- Use short, punchy sentences most of the time.
- Vary the rhythm. Short punch. Then a little more detail when it helps.
- Use casual connectors when they fit: "so", "anyway", "plus", "also".
- Be honest when something is weird, unclear, or unknown.
- Use light natural hesitation sparingly: "I think maybe", "probably", "not sure but".
- Avoid marketing speak, corporate buzzwords, stiff transitions, and obvious AI phrases like "dive into", "delve into", or "let's explore".
- Avoid piling on adjectives.
- Keep capitalization normal.
- Be relaxed, but still clear. Real, not sloppy.

## Read this first

0. `AGENT_START.md` for the short agent entrypoint
1. `README.md`
2. `AGENTS.md`
3. `docs/repo-layout.md`
4. `docs/agent-onboarding.md`
5. `CLAUDE.md`
6. `Sources/CLAUDE.md`
7. the nearest local `CLAUDE.md` for the area you are changing
8. `Sources/Accessibility/CLAUDE.md` when touching focused-editor AX metadata, overlay placement, or paste-back context
9. `Sources/Beta/CLAUDE.md` when touching beta-build configuration
10. `Sources/Dictation/CLAUDE.md` when touching dictation persistence
11. `Sources/Meeting/CLAUDE.md` when touching meeting capture or meeting UI
12. `Sources/TranscriptedCore/CLAUDE.md` when touching the shared library
13. `Sources/Speech/CLAUDE.md` when touching dictation STT, audio recovery, or device handling
14. `Sources/Support/CLAUDE.md` when touching shared preferences, permissions, paths, or Claude Desktop install flow
15. `Sources/UI/CLAUDE.md` when touching overlay, menubar, onboarding, settings, or agent-connect UI
16. `Sources/Capture/CLAUDE.md` when touching hotkeys or physical dictation trigger routing
17. `Tests/README.md`
18. `docs/storage-paths.md`
19. `Sources/Reliability/CLAUDE.md` when touching wake / sleep recovery or hotkey recovery
20. `Sources/Observability/CLAUDE.md` when touching crash reporting, event forwarding, anonymous analytics, or app updates
21. `docs/release-packaging.md` when touching packaging, signing, notarization, or user-facing releases
22. `docs/sparkle-updates.md` when touching app updates or cutting a release users should receive in-app
23. `Tools/*/CLAUDE.md` when touching standalone CLI, MCP, or QA tools

Use `docs/repo-layout.md` as the canonical directory map and doc hierarchy.
Use `.agents/test-matrix.yml` as the quick path-to-verification map, with this
file taking precedence when there is any conflict.

## Build and test

```bash
bash build-deps.sh
bash build.sh
bash run-tests.sh
bash run-integration-smoke.sh
swift test
```

Rules:

1. After changing Swift source, run `bash build.sh` and `bash run-tests.sh`.
2. If you touch `Sources/Meeting/` or `Sources/TranscriptedCore/`, also run `bash run-integration-smoke.sh`.
3. If you touch `Package.swift`, `Sources/TranscriptedCore/`, or the public core seam, also run `swift test`.
4. `build.sh` must not compile `Sources/TranscriptedCore/` directly into the app target.

## Releases, Sparkle, and Homebrew

When the task is a user-facing release, package handoff, or update-path change,
agents must treat Sparkle as part of the release contract, not as optional
follow-up work.

Rules:

1. Read `docs/release-packaging.md` and `docs/sparkle-updates.md` before changing release flow.
2. For builds intended for other machines, use `build-beta.sh`, not `build.sh`.
3. A release is not complete just because a DMG exists. For in-app updates to work, the release flow must also:
   - publish the signed archive where users can fetch it
   - update `docs/appcast.xml`
   - push the updated appcast to the branch that backs the live feed
4. If the release should also be installable or upgradeable through Homebrew, the release flow must also:
   - run `bash scripts/release/update-cask.sh <version>` after the GitHub release is published
   - commit the updated `Casks/transcripted.rb`
   - push that cask update so `brew install` and `brew upgrade` see the new version
5. If Sparkle metadata was not updated, say explicitly that existing installs will not discover the new release in-app yet.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [r3dbars/transcripted](https://github.com/r3dbars/transcripted) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
